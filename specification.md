This specification describes the functional requirements, architecture considerations, data handling details, error handling approaches, and a testing plan for creating a single-page web app that takes an image, obtains bounding box data from an AI (LLM or computer vision model), and validates the returned data against a custom JSON Schema.

---

# **1\. Overview**

You want a single-page HTML web application (SPA) that:

1. Lets a user upload or provide an image.  
2. Sends the image to one or more Large Language Models (LLMs) or computer vision services (collectively, “the AI”) for analysis.  
3. Receives a JSON response describing bounding boxes within the image.  
4. Validates the JSON response against a *custom bounding box schema* (which enforces normalized coordinates, mandatory labels with confidence scores, etc.).  
5. Displays the validation results (pass/fail, any errors) and potentially the bounding box overlays on the image for manual inspection.

## **1.1 Primary Use Case**

* **Goal**: Evaluate and compare how different LLMs/CV models perform image analysis tasks by returning bounding boxes (along with descriptive labels).  
* **Scenario**: The user uploads an image (e.g., “excited dog running in grass”). The system calls the AI, which returns bounding box data in a JSON structure conforming to the schema below, then checks for schema validity and presents the results.

---

# **2\. Requirements**

1. **Single-Page Interface**

   * A minimal HTML page with embedded JavaScript.  
   * File/image upload functionality (or drag-and-drop).  
   * A place to show the validated JSON output and/or any error messages.  
2. **Image Upload and AI Request**

   * The user selects an image (JPEG, PNG, etc.).  
   * The app sends this image (or a reference/URL to it) to an AI service endpoint.  
     * **Note**: The actual AI service integration is pluggable; we may have multiple endpoints to test different models.  
3. **JSON Schema Validation**

   * The AI responds with JSON describing bounding boxes.  
   * The app validates that JSON against the agreed-upon schema.  
   * The result of the validation is displayed to the user (success, or a list of validation errors).  
4. **Custom Bounding Box Schema**

   * Must require `width`, `height`, and `bounding_boxes` in `image`.  
   * `width`/`height`: integers.  
   * `bounding_boxes`: an array of 0 or more bounding box objects.  
     * Each bounding box has `x_min`, `y_min`, `x_max`, `y_max` in `[0, 1]`.  
     * Each box must have a non-empty `labels` array.  
       * Each label has `name` (string) and `confidence` (integer in `[0, 100]`).  
5. **Flexibility and Extensibility**

   * The schema allows additional properties beyond those required.  
   * The user might pass extra data, but the core required fields must conform to the schema.  
6. **Usability**

   * The page should make it clear if the response is valid or not.  
   * Potentially show bounding boxes visually on the uploaded image if validated (this is optional but desirable).

---

# **3\. Proposed Architecture**

A straightforward approach is a **Front-End Only** SPA that:

1. **HTML \+ JavaScript**:  
   * Renders a form for selecting an image.  
   * On form submission, calls a JavaScript function.  
2. **AI Integration**:  
   * Possibly uses `fetch` or `XMLHttpRequest` to send the image or its base64 string to a back-end AI endpoint (could be a microservice, serverless function, or external API).  
   * Receives a JSON response with bounding box data.  
3. **JSON Validation**:  
   * Use a JSON Schema validator library (e.g., [Ajv](https://ajv.js.org/) if using JavaScript) in the browser.  
   * Validate the AI’s JSON against the custom bounding box schema.  
   * Display results.  
4. **Visualization (optional but recommended)**:  
   * Draw bounding boxes on a canvas overlay or using HTML elements (if the user wants a direct visual check).

### **3.1 Alternative: Back-End \+ Front-End**

If the AI call or validation is too large/complex for the front-end, a minimal Node.js/Express back end could:

* Receive the uploaded image from the front-end.  
* Forward it to the AI service.  
* Validate the response with a Node-based JSON Schema library.  
* Send the validated result back to the client.

**Either approach** is viable. The key requirement is to do JSON Schema validation to ensure compliance.

---

# **4\. Data Handling Workflow**

1. **User Action**  
   * The user selects an image file from their device (JPEG, PNG, etc.).  
2. **Front-End Processing**  
   * Option A: Convert image to base64 or FormData, then send it to the AI service endpoint.  
   * Option B: Provide an image URL if the image is hosted somewhere.  
3. **AI Response**

The AI returns a JSON structure that *should* look like:  
 {  
  "image": {  
    "width": 1024,  
    "height": 768,  
    "bounding\_boxes": \[  
      {  
        "x\_min": 0.1,  
        "y\_min": 0.2,  
        "x\_max": 0.3,  
        "y\_max": 0.4,  
        "labels": \[  
          { "name": "dog", "confidence": 95 },  
          { "name": "running", "confidence": 80 }  
        \]  
      }  
      // ... more bounding boxes ...  
    \]  
  }  
}

*   
4. **Validation**  
   * Use the JSON Schema (see Section 6\) to validate the response.  
   * If it’s valid, proceed. If not, display errors.  
5. **Optional Display**  
   * Render bounding boxes on the image if desired.  
   * Let the user see the labeled objects visually.

---

# **5\. Error Handling Strategy**

1. **AI Call Failures**  
   * If the AI endpoint is unreachable or returns an error code (4xx/5xx), show a clear message (“Unable to process image. Please try again.”).  
2. **Malformed JSON**  
   * If the response is not valid JSON, catch the parsing error and notify the user.  
3. **Schema Validation Failures**  
   * If any required fields are missing or incorrectly typed, show a structured list of validation errors. For instance:  
     * `"image.width is required and must be an integer"`  
     * `"bounding_boxes[0].labels must contain at least one item"`, etc.  
4. **Coordinate Constraints**  
   * The schema ensures numeric values for the bounding boxes. If `x_min`, `y_min`, etc. is outside `[0,1]`, you’ll get a validation error.  
   * The user is informed that the bounding box data is invalid if that happens.  
5. **Confidence Score Issues**  
   * If `confidence` is not an integer in `[0, 100]`, the validator flags it.

---

# **6\. JSON Schema (Draft 2020-12)**

Below is the final schema you decided on. Store it in your project (e.g., `bbox-schema.json`) and reference it for validation.

{  
  "$schema": "https://json-schema.org/draft/2020-12/schema",  
  "$id": "https://example.com/my-bbox-schema.json",  
  "type": "object",  
  "title": "Image Bounding Box Schema",  
  "properties": {  
    "image": {  
      "type": "object",  
      "properties": {  
        "width": {  
          "type": "integer",  
          "minimum": 1  
        },  
        "height": {  
          "type": "integer",  
          "minimum": 1  
        },  
        "bounding\_boxes": {  
          "type": "array",  
          "items": {  
            "type": "object",  
            "properties": {  
              "x\_min": {  
                "type": "number",  
                "minimum": 0,  
                "maximum": 1  
              },  
              "y\_min": {  
                "type": "number",  
                "minimum": 0,  
                "maximum": 1  
              },  
              "x\_max": {  
                "type": "number",  
                "minimum": 0,  
                "maximum": 1  
              },  
              "y\_max": {  
                "type": "number",  
                "minimum": 0,  
                "maximum": 1  
              },  
              "labels": {  
                "type": "array",  
                "minItems": 1,  
                "items": {  
                  "type": "object",  
                  "properties": {  
                    "name": {  
                      "type": "string"  
                    },  
                    "confidence": {  
                      "type": "integer",  
                      "minimum": 0,  
                      "maximum": 100  
                    }  
                  },  
                  "required": \["name", "confidence"\],  
                  "additionalProperties": true  
                }  
              }  
            },  
            "required": \["x\_min", "y\_min", "x\_max", "y\_max", "labels"\],  
            "additionalProperties": true  
          }  
        }  
      },  
      "required": \["width", "height", "bounding\_boxes"\],  
      "additionalProperties": true  
    }  
  },  
  "required": \["image"\],  
  "additionalProperties": true  
}

### **Validation Library Options**

* **Front End**: [Ajv](https://ajv.js.org/) is a popular choice.  
* **Back End** (Node.js, Python, etc.): Many JSON Schema libraries exist; choose one compatible with Draft 2020-12.

---

# **7\. Testing Plan**

The testing strategy should cover:

1. **Unit Tests**

   * **Schema Validation**: Feed known valid JSON and verify no errors. Feed invalid JSON (missing fields, out-of-range values, etc.) and confirm errors are triggered.  
   * **UI Elements**: Ensure the file upload, response display, and error messages behave as expected in isolation.  
2. **Integration Tests**

   * **AI Endpoint Mock**: Create a mock or stub AI service that returns a valid bounding box JSON. Then test how your app handles that response.  
   * **Edge Cases**:  
     * Zero bounding boxes.  
     * Multiple bounding boxes.  
     * Large images with bounding boxes close to the edges (e.g., `x_min = 0`, `x_max = 1`).  
     * Confidence \= 0 or 100\.  
     * Malformed JSON or incorrect fields.  
   * **Network Errors**: Simulate timeouts or 500 errors from the AI service.  
3. **End-to-End User Flow**

   * Launch the SPA, pick an image, verify a valid response is displayed properly, see bounding boxes on the image (if you implement that), confirm no console errors.  
4. **Performance Testing** (Optional)

   * If you expect large images or high traffic, measure how quickly the AI call \+ validation completes. Possibly compress images for speed.  
5. **Cross-Browser Testing**

   * Validate in modern browsers (Chrome, Firefox, Safari, Edge).  
   * Confirm front-end validations and displays look consistent.

---

# **8\. Development & Next Steps**

1. **Initial Setup**  
   * Build a minimal HTML/JavaScript page.  
   * Integrate a JSON Schema validator library (like Ajv).  
2. **File Upload**  
   * Provide an `<input type="file" />` or drag-and-drop area.  
3. **AI Communication**  
   * Decide on how to send images (FormData, base64, or direct link).  
   * Possibly store configuration for multiple AI endpoints to compare results from each.  
4. **Validation & Display**  
   * After receiving the AI’s JSON, run the validator.  
   * If valid, show “Success\!” Possibly overlay bounding boxes.  
   * If invalid, show error details.  
5. **Optional Enhancements**  
   * Display bounding box overlays with label text and confidence.  
   * Add user feedback loops (e.g., rating the bounding box quality).  
   * Store or export results for further analysis.

With these details, a developer has everything needed to implement the single-page application:

* **Clear data format** (the JSON Schema).  
* **Core UI flow** (upload → send → receive → validate → display results).  
* **Error handling** guidelines and test coverage.

This specification is flexible enough to allow different AI back-end services and additional front-end features (like bounding box visualization) without breaking schema compliance.