# AWS Bedrock Prompt Router Usage

## Overview

AWS Bedrock's Prompt Router capability allows for intelligent routing between models from the same provider. This feature is particularly useful when you want to leverage the strengths of multiple models without manually selecting each one.

## Supported Prompt Routers

### Anthropic Prompt Router
- Routes between Claude 3 Haiku and Claude 3.5 Sonnet

### Meta Prompt Router
- Routes between Llama 3.1 8B Instruct and Llama 3.1 70B Instruct

## When to Use Prompt Router

### Use Cases
- **Dynamic Model Selection**: Automatically select the best model based on the input prompt.
- **Cost Optimization**: Use less expensive models for simpler tasks and more powerful models for complex tasks.
- **Performance Optimization**: Improve response times by routing to the most efficient model.

### Examples
- **Anthropic Prompt Router**: Ideal for tasks requiring a balance between creativity and precision.
- **Meta Prompt Router**: Suitable for tasks that need a mix of instruction-following and general-purpose capabilities.

## Configuration

### Anthropic Prompt Router

```json
{
  "apiProvider": "bedrock",
  "apiModelId": "anthropic-prompt-router",
  "awsAccessKey": "your-access-key",
  "awsSecretKey": "your-secret-key",
  "awsRegion": "your-region"
}
```

### Meta Prompt Router

```json
{
  "apiProvider": "bedrock",
  "apiModelId": "meta-prompt-router",
  "awsAccessKey": "your-access-key",
  "awsSecretKey": "your-secret-key",
  "awsRegion": "your-region"
}
```

## UI Configuration

1. Open the settings panel in the application.
2. Navigate to the model dropdown.
3. Select either "Anthropic Prompt Router" or "Meta Prompt Router".
4. Save the configuration.

## API Integration

Ensure that your API calls are updated to support the Prompt Router endpoints. This involves setting the `apiModelId` to either `anthropic-prompt-router` or `meta-prompt-router` in your configuration.

## Error Handling

Implement error handling for router-specific failures. This includes:
- Validating router-specific configurations.
- Handling different pricing structures.
- Implementing a caching strategy for router decisions.

## Testing

### Unit Tests
- Add unit tests for Anthropic Prompt Router and Meta Prompt Router in `AwsBedrockHandler`.

### Integration Tests
- Add integration tests with AWS Bedrock Prompt Router API.

### UI Tests
- Add UI tests for new dropdown options.

## Conclusion

By leveraging AWS Bedrock's Prompt Router capability, you can optimize model selection dynamically, improve performance, and reduce costs. Follow the guidelines and examples provided to integrate and configure the Prompt Router in your application.
