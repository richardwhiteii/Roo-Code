# MCP Documentation

## Overview of the MCP Feature

The Model Context Protocol (MCP) is a powerful feature that enables communication between the system and locally running MCP servers. These servers provide additional tools and resources to extend your capabilities. MCP servers can be used to interact with external systems, perform computations, and take actions in the real world.

## Creating New MCP Servers

To create a new MCP server, follow these steps:

1. **Bootstrap a new project**: Use the `create-typescript-server` tool to bootstrap a new project in the default MCP servers directory.

    ```bash
    cd /path/to/mcp-servers
    npx @modelcontextprotocol/create-server my-server
    cd my-server
    npm install
    ```

2. **Implement the server**: Replace the contents of `src/index.ts` with your server implementation. Here is an example of a simple MCP server that provides weather data using the OpenWeather API:

    ```typescript
    #!/usr/bin/env node
    import { Server } from '@modelcontextprotocol/sdk/server/index.js';
    import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
    import {
      CallToolRequestSchema,
      ErrorCode,
      ListResourcesRequestSchema,
      ListResourceTemplatesRequestSchema,
      ListToolsRequestSchema,
      McpError,
      ReadResourceRequestSchema,
    } from '@modelcontextprotocol/sdk/types.js';
    import axios from 'axios';

    const API_KEY = process.env.OPENWEATHER_API_KEY; // provided by MCP config
    if (!API_KEY) {
      throw new Error('OPENWEATHER_API_KEY environment variable is required');
    }

    interface OpenWeatherResponse {
      main: {
        temp: number;
        humidity: number;
      };
      weather: [{ description: string }];
      wind: { speed: number };
      dt_txt?: string;
    }

    const isValidForecastArgs = (
      args: any
    ): args is { city: string; days?: number } =>
      typeof args === 'object' &&
      args !== null &&
      typeof args.city === 'string' &&
      (args.days === undefined || typeof args.days === 'number');

    class WeatherServer {
      private server: Server;
      private axiosInstance;

      constructor() {
        this.server = new Server(
          {
            name: 'example-weather-server',
            version: '0.1.0',
          },
          {
            capabilities: {
              resources: {},
              tools: {},
            },
          }
        );

        this.axiosInstance = axios.create({
          baseURL: 'http://api.openweathermap.org/data/2.5',
          params: {
            appid: API_KEY,
            units: 'metric',
          },
        });

        this.setupResourceHandlers();
        this.setupToolHandlers();
        
        // Error handling
        this.server.onerror = (error) => console.error('[MCP Error]', error);
        process.on('SIGINT', async () => {
          await this.server.close();
          process.exit(0);
        });
      }

      // MCP Resources represent any kind of UTF-8 encoded data that an MCP server wants to make available to clients, such as database records, API responses, log files, and more. Servers define direct resources with a static URI or dynamic resources with a URI template that follows the format `[protocol]://[host]/[path]`.
      private setupResourceHandlers() {
        // For static resources, servers can expose a list of resources:
        this.server.setRequestHandler(ListResourcesRequestSchema, async () => ({
          resources: [
            // This is a poor example since you could use the resource template to get the same information but this demonstrates how to define a static resource
            {
              uri: `weather://San Francisco/current`, // Unique identifier for San Francisco weather resource
              name: `Current weather in San Francisco`, // Human-readable name
              mimeType: 'application/json', // Optional MIME type
              // Optional description
              description:
                'Real-time weather data for San Francisco including temperature, conditions, humidity, and wind speed',
            },
          ],
        }));

        // For dynamic resources, servers can expose resource templates:
        this.server.setRequestHandler(
          ListResourceTemplatesRequestSchema,
          async () => ({
            resourceTemplates: [
              {
                uriTemplate: 'weather://{city}/current', // URI template (RFC 6570)
                name: 'Current weather for a given city', // Human-readable name
                mimeType: 'application/json', // Optional MIME type
                description: 'Real-time weather data for a specified city', // Optional description
              },
            ],
          })
        );

        // ReadResourceRequestSchema is used for both static resources and dynamic resource templates
        this.server.setRequestHandler(
          ReadResourceRequestSchema,
          async (request) => {
            const match = request.params.uri.match(
              /^weather:\/\/([^/]+)\/current$/
            );
            if (!match) {
              throw new McpError(
                ErrorCode.InvalidRequest,
                `Invalid URI format: ${request.params.uri}`
              );
            }
            const city = decodeURIComponent(match[1]);

            try {
              const response = await this.axiosInstance.get(
                'weather', // current weather
                {
                  params: { q: city },
                }
              );

              return {
                contents: [
                  {
                    uri: request.params.uri,
                    mimeType: 'application/json',
                    text: JSON.stringify(
                      {
                        temperature: response.data.main.temp,
                        conditions: response.data.weather[0].description,
                        humidity: response.data.main.humidity,
                        wind_speed: response.data.wind.speed,
                        timestamp: new Date().toISOString(),
                      },
                      null,
                      2
                    ),
                  },
                ],
              };
            } catch (error) {
              if (axios.isAxiosError(error)) {
                throw new McpError(
                  ErrorCode.InternalError,
                  `Weather API error: ${
                    error.response?.data.message ?? error.message
                  }`
                );
              }
              throw error;
            }
          }
        );
      }

      /* MCP Tools enable servers to expose executable functionality to the system. Through these tools, you can interact with external systems, perform computations, and take actions in the real world.
       * - Like resources, tools are identified by unique names and can include descriptions to guide their usage. However, unlike resources, tools represent dynamic operations that can modify state or interact with external systems.
       * - While resources and tools are similar, you should prefer to create tools over resources when possible as they provide more flexibility.
       */
      private setupToolHandlers() {
        this.server.setRequestHandler(ListToolsRequestSchema, async () => ({
          tools: [
            {
              name: 'get_forecast', // Unique identifier
              description: 'Get weather forecast for a city', // Human-readable description
              inputSchema: {
                // JSON Schema for parameters
                type: 'object',
                properties: {
                  city: {
                    type: 'string',
                    description: 'City name',
                  },
                  days: {
                    type: 'number',
                    description: 'Number of days (1-5)',
                    minimum: 1,
                    maximum: 5,
                  },
                },
                required: ['city'], // Array of required property names
              },
            },
          ],
        }));

        this.server.setRequestHandler(CallToolRequestSchema, async (request) => {
          if (request.params.name !== 'get_forecast') {
            throw new McpError(
              ErrorCode.MethodNotFound,
              `Unknown tool: ${request.params.name}`
            );
          }

          if (!isValidForecastArgs(request.params.arguments)) {
            throw new McpError(
              ErrorCode.InvalidParams,
              'Invalid forecast arguments'
            );
          }

          const city = request.params.arguments.city;
          const days = Math.min(request.params.arguments.days || 3, 5);

          try {
            const response = await this.axiosInstance.get<{
              list: OpenWeatherResponse[];
            }>('forecast', {
              params: {
                q: city,
                cnt: days * 8,
              },
            });

            return {
              content: [
                {
                  type: 'text',
                  text: JSON.stringify(response.data.list, null, 2),
                },
              ],
            };
          } catch (error) {
            if (axios.isAxiosError(error)) {
              return {
                content: [
                  {
                    type: 'text',
                    text: `Weather API error: ${
                      error.response?.data.message ?? error.message
                    }`,
                  },
                ],
                isError: true,
              };
            }
            throw error;
          }
        });
      }

      async run() {
        const transport = new StdioServerTransport();
        await this.server.connect(transport);
        console.error('Weather MCP server running on stdio');
      }
    }

    const server = new WeatherServer();
    server.run().catch(console.error);
    ```

3. **Build and compile the executable JavaScript file**:

    ```bash
    npm run build
    ```

4. **Configure the MCP server**: Add the MCP server configuration to the settings file located at `/path/to/mcp-settings.json`. The settings file may have other MCP servers already configured, so you would read it first and then add your new server to the existing `mcpServers` object.

    ```json
    {
      "mcpServers": {
        ...,
        "weather": {
          "command": "node",
          "args": ["/path/to/weather-server/build/index.js"],
          "env": {
            "OPENWEATHER_API_KEY": "user-provided-api-key"
          }
        }
      }
    }
    ```

5. **Run the MCP server**: The system will automatically run all the servers and expose the available tools and resources in the 'Connected MCP Servers' section.

## Using MCP Servers

Once an MCP server is connected, you can use the server's tools and resources. Here are some common commands and configurations:

- **List available tools**: Use the `tools/list` method to get a list of tools provided by the server.
- **Call a tool**: Use the `tools/call` method to execute a tool provided by the server. Provide the tool name and input parameters as required by the tool's input schema.
- **List resources**: Use the `resources/list` method to get a list of resources provided by the server.
- **Read a resource**: Use the `resources/read` method to read the contents of a resource provided by the server.

## Connecting to External MCP Servers

To connect to external MCP servers running outside of the codebase, follow these guidelines:

1. **Obtain the server's configuration**: Get the server's command, arguments, and environment variables required to run the server.
2. **Add the server configuration**: Add the server configuration to the settings file located at `/path/to/mcp-settings.json`.
3. **Run the server**: The system will automatically run the external server and expose its tools and resources.

### Example

Here is an example of how to connect to an external MCP server:

1. **Obtain the server's configuration**:

    ```json
    {
      "command": "node",
      "args": ["/path/to/external-server/build/index.js"],
      "env": {
        "EXTERNAL_API_KEY": "external-api-key"
      }
    }
    ```

2. **Add the server configuration**: Add the server configuration to the settings file located at `/path/to/mcp-settings.json`.

    ```json
    {
      "mcpServers": {
        ...,
        "external-server": {
          "command": "node",
          "args": ["/path/to/external-server/build/index.js"],
          "env": {
            "EXTERNAL_API_KEY": "external-api-key"
          }
        }
      }
    }
    ```

3. **Run the server**: The system will automatically run the external server and expose its tools and resources.

By following these steps, you can create new MCP servers, use MCP servers, and connect to external MCP servers to extend your system's capabilities.
