# VisionValidator

VisionValidator is a single-page web application that evaluates and compares how different AI vision models perform object detection tasks by returning bounding box data.

## Overview

This tool helps developers, researchers and AI enthusiasts:

- Test various AI vision models with your own images
- Validate the returned bounding box data against a schema
- Visualize object detection results with bounding boxes
- Compare consistency across multiple queries with heat maps
- Analyze contextual attributes identified in the image

## Features

- **Multi-Model Support**: Works with OpenAI GPT-4o, Google Gemini, and Anthropic Claude 3
- **Bounding Box Visualization**: Displays detected objects with colored bounding boxes
- **Heat Map Generation**: Creates visual representations of AI consistency across multiple queries
- **Schema Validation**: Ensures AI responses conform to a standardized format
- **Secure API Key Management**: Stores API keys locally in your browser
- **Multiple Query Analysis**: Makes 5 identical requests to analyze model consistency
- **Response Comparison**: Toggle between different AI responses for the same image

## Getting Started

### Usage

1. **Select an AI Model**: Choose between OpenAI GPT-4o, Google Gemini, or Anthropic Claude 3
2. **Enter Your API Key**: Provide your API key for the selected service (stored locally in your browser)
3. **Upload an Image**: Select any image you wish to analyze
4. **Process the Image**: Click "Send to AI & Validate" to start analysis
5. **View Results**: 
   - See bounding boxes drawn on your image
   - View the heat map showing detection consistency
   - Toggle between different AI responses using the selector
   - Review the full JSON response below

### Requirements

- Modern web browser (Chrome, Firefox, Safari, Edge)
- API key for at least one of the supported AI services:
  - [OpenAI API key](https://platform.openai.com/api-keys)
  - [Google AI API key](https://aistudio.google.com/app/apikey)
  - [Anthropic API key](https://console.anthropic.com/keys)

## Technical Details

VisionValidator is a frontend-only application built with:

- HTML, JavaScript, and Tailwind CSS
- LangChain.js for AI model integration
- Ajv for JSON schema validation
- Canvas API for visualization

### Privacy Considerations

- All processing happens in your browser
- Images never touch our servers - they're sent directly from your browser to the AI provider
- API keys are stored in your browser's localStorage and only sent to their respective providers

## Examples

### Use Cases

- **AI Research**: Evaluate and compare different vision models
- **Quality Assurance**: Test consistency of object detection across multiple queries
- **Model Evaluation**: Determine which AI performs best for specific image types
- **Educational Tool**: Learn about how AI systems interpret visual information

## Contributing

Contributions are welcome! Feel free to submit issues or pull requests.