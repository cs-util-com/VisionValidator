This specification describes the functional requirements, architecture considerations, data handling details, error handling approaches, and a testing plan for creating a single-page web app that takes an image, obtains bounding box data from an AI (LLM or computer vision model), and validates the returned data against a custom JSON Schema.

---

# **1\. Overview**

You want a single-page HTML web application (SPA) that:

1. Lets a user upload or provide an image.  
2. Sends the image to one or more Large Language Models (LLMs) or computer vision services (collectively, "the AI") for analysis.  
3. Receives a JSON response describing bounding boxes within the image.  
4. Validates the JSON response against a *custom bounding box schema* (which enforces normalized coordinates, mandatory labels with confidence scores, etc.).  
5. Displays the validation results (pass/fail, any errors) and potentially the bounding box overlays on the image for manual inspection.
6. Supports multiple AI models (OpenAI GPT-4o, Google Gemini, Anthropic Claude 3).
7. Creates heat map visualizations that aggregate results from multiple AI queries.

## **1.1 Primary Use Case**

* **Goal**: Evaluate and compare how different LLMs/CV models perform image analysis tasks by returning bounding boxes (along with descriptive labels).  
* **Scenario**: The user uploads an image (e.g., "excited dog running in grass"). The system calls the AI, which returns bounding box data in a JSON structure conforming to the schema below, then checks for schema validity and presents the results.
* **Comparison**: The app generates multiple responses from the same AI model to analyze consistency and renders an aggregated heat map visualization.

---

# **2\. Requirements**

1. **Single-Page Interface**

   * A minimal HTML page with embedded JavaScript.  
   * File/image upload functionality (or drag-and-drop).  
   * A place to show the validated JSON output and/or any error messages.
   * Model selector dropdown for choosing different AI providers.
   * API key input field that stores keys locally in the browser.
   * Response selector to view individual AI responses.
   
2. **Image Upload and AI Request**

   * The user selects an image (JPEG, PNG, etc.).  
   * The app sends this image (or a reference/URL to it) to an AI service endpoint. 
   * Supports OpenAI GPT-4o, Google Gemini, and Anthropic Claude 3 models.
   * Makes multiple queries (5 by default) to the same model for consistency analysis.
     
3. **JSON Schema Validation**

   * The AI responds with JSON describing bounding boxes.  
   * The app validates that JSON against the agreed-upon schema using Ajv.  
   * The result of the validation is displayed to the user (success, or a list of validation errors).  
   
4. **Custom Bounding Box Schema**

   * Must require `width`, `height`, `bounding_boxes` and `contextual_attributes` in `image`.  
   * `width`/`height`: integers.  
   * `bounding_boxes`: an array of 0 or more bounding box objects.  
     * Each bounding box has `x_min`, `y_min`, `x_max`, `y_max` as integer pixel coordinates.  
     * Each box must have a non-empty `labels` array.  
       * Each label has `name` (string) and `confidence` (integer in `[0, 100]`).  
   * `contextual_attributes`: an array of objects with `attribute` and `description` strings.
   
5. **Visualization Features**

   * Bounding boxes displayed as overlays on the image with different colors.
   * Heat map visualization showing aggregated results from multiple queries.
   * Ability to switch between individual AI responses for comparison.
   * Labels displayed next to bounding boxes with confidence scores.
   
6. **Flexibility and Extensibility**

   * The schema allows additional properties beyond those required.  
   * The user might pass extra data, but the core required fields must conform to the schema.  
   
7. **API Key Management**
   
   * API keys stored locally in browser's localStorage.
   * Different key fields for each supported AI provider.
   * Dynamic labeling based on selected model.

---

# **3\. Implemented Architecture**

A **Front-End Only** SPA that:

1. **HTML \+ JavaScript**:  
   * Renders a form for selecting an image and AI model.
   * Manages API keys for different providers.
   * On form submission, calls the selected AI service.
   
2. **AI Integration**:  
   * Uses the LangChain library for standardized integration with multiple AI providers.
   * Sends the image as base64 to the selected AI model.
   * Makes multiple queries (default: 5) for consistency analysis.
   * Receives JSON responses with bounding box data.
   
3. **JSON Validation**:  
   * Uses Ajv for JSON Schema validation.
   * Validates the AI's JSON against the custom bounding box schema.
   * Displays validation results.
   
4. **Visualization**:  
   * Draws bounding boxes with labels on a canvas overlay.
   * Generates heat map visualizations showing consistency across multiple responses.
   * Applies color coding for different object types.

---

# **4\. Data Handling Workflow**

1. **User Action**  
   * The user selects an AI model and provides their API key.
   * The user selects an image file from their device.
   
2. **Front-End Processing**  
   * Converts image to base64 and sends it to the selected AI service.
   * Makes multiple identical queries to analyze consistency.
   
3. **AI Response**

The AI returns JSON structures that should conform to this schema:  
```json
{  
  "image": {  
    "width": 1024,  
    "height": 768,  
    "bounding_boxes": [  
      {  
        "x_min": 200,  
        "y_min": 100,  
        "x_max": 400,  
        "y_max": 300,  
        "labels": [  
          { "name": "dog", "confidence": 95 },  
          { "name": "running", "confidence": 80 }  
        ]  
      }  
      // ... more bounding boxes ...  
    ],
    "contextual_attributes": [
      { "attribute": "weather", "description": "Sunny day" },
      { "attribute": "location", "description": "Public park" }
    ]  
  }  
}
```

4. **Validation**  
   * Uses the JSON Schema to validate each response.
   * Filters out invalid responses.
   
5. **Visualization**  
   * Renders individual bounding boxes for each valid response.
   * Generates a heat map overlay showing agreement across multiple queries.
   * Allows switching between individual responses.

---

# **5\. Error Handling Strategy**

1. **AI Call Failures**  
   * If the AI endpoint is unreachable or returns an error code, skips that response.
   * Continues processing other successful responses.
   
2. **Malformed JSON**  
   * Attempts to clean JSON from markdown code blocks if needed.
   * If the response is not valid JSON, skips that response.
   
3. **Schema Validation Failures**  
   * If the JSON doesn't validate against the schema, logs errors.
   * Only processes valid responses for visualization.
   
4. **Partial Success Strategy**
   * If some queries succeed and others fail, works with the successful ones.
   * Shows a count of successful vs. total queries.

---

# **6\. JSON Schema**

The implemented schema includes support for bounding boxes and contextual attributes:

```json
{
  "$id": "https://example.com/my-bbox-schema.json",
  "type": "object",
  "title": "Image Bounding Box Schema",
  "properties": {
    "image": {
      "type": "object",
      "properties": {
        "width": { "type": "integer", "minimum": 1 },
        "height": { "type": "integer", "minimum": 1 },
        "bounding_boxes": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "x_min": { "type": "integer", "minimum": 0 },
              "y_min": { "type": "integer", "minimum": 0 },
              "x_max": { "type": "integer", "minimum": 0 },
              "y_max": { "type": "integer", "minimum": 0 },
              "labels": {
                "type": "array",
                "minItems": 1,
                "items": {
                  "type": "object",
                  "properties": {
                    "name": { "type": "string" },
                    "confidence": { "type": "integer", "minimum": 0, "maximum": 100 }
                  },
                  "required": ["name", "confidence"],
                  "additionalProperties": true
                }
              }
            },
            "required": ["x_min", "y_min", "x_max", "y_max", "labels"],
            "additionalProperties": true
          }
        },
        "contextual_attributes": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "attribute": { "type": "string" },
              "description": { "type": "string" }
            },
            "required": ["attribute", "description"],
            "additionalProperties": true
          }
        }
      },
      "required": ["width", "height", "bounding_boxes", "contextual_attributes"],
      "additionalProperties": true
    }
  },
  "required": ["image"],
  "additionalProperties": true
}
```

### **Validation Library**

* Using **Ajv** for JSON Schema validation with specific options to handle potential issues:
  ```javascript
  const ajv = new Ajv({
    strict: false,
    strictSchema: false,
    strictTypes: false,
    validateSchema: false
  });
  ```

---

# **7\. Heat Map Implementation**

The application implements a heat map visualization that:

1. Makes multiple queries (5 by default) to the same AI model.
2. Aggregates responses into a grid-based heat map.
3. Applies a Gaussian blur for smoother visualization.
4. Uses different colors for different object labels.
5. Shows transparency based on confidence/agreement level.

This helps visualize:
* Areas where the AI consistently detects objects
* Differences in boundary precision across queries
* Confidence levels for different parts of the image

---

# **8\. Multi-Model Support**

The application supports these AI models:

1. **OpenAI GPT-4o** - Using the ChatOpenAI class from LangChain
2. **Google Gemini** - Using the ChatGoogleGenerativeAI class from LangChain
3. **Anthropic Claude 3** - Using the ChatAnthropic class from LangChain

Each model requires its own API key, which is:
* Stored securely in the browser's localStorage
* Not transmitted except to the respective AI provider
* Referenced by a unique key based on the model name

---

# **9\. Testing & Future Enhancements**

1. **Testing Considerations**
   * Test with various image types and sizes
   * Compare responses across different AI models
   * Validate heat map generation with different sample counts
   
2. **Potential Enhancements**
   * Save/export analyzed images with bounding boxes
   * Add additional AI model providers
   * Include confidence thresholds for filtering results
   * Implement side-by-side comparison between models
   * Add annotation tools to correct or refine AI-generated boxes