# Guide: Creating an Image Compression API with n8n

This guide provides a step-by-step walkthrough for turning your client-side `Compress` application into a powerful, automated backend API using n8n.

The core idea is to use n8n to wrap your compression logic in a simple webhook, making it available as a service to any other application.

## Prerequisites

1.  **A running n8n instance:** You should have n8n installed and running. You can refer to the `n8n_digital_ocean_setup.md` guide for instructions.
2.  **Node.js Compression Logic:** The image compression logic from your `Compress/script.js` file needs to be adaptable to a Node.js environment. Client-side JavaScript libraries for things like DOM manipulation will not work. For robust server-side image processing in Node.js, a library like `sharp` is highly recommended. You may need to add this library to your n8n instance.

---

## Workflow Setup

This workflow requires only two nodes: a **Webhook** to receive the image and a **Code** node to process it.

### Part 1: The Webhook Trigger

The Webhook node will be the public API endpoint for your service.

1.  In your n8n canvas, create a new workflow.
2.  Add a new **Webhook** node. This will be your trigger.
3.  In the Webhook node's settings panel on the right:
    *   **HTTP Method:** Set this to `POST`.
    *   **Binary Data:** Set this to `On`. This is crucial for allowing the webhook to accept file uploads.
    *   **Path:** The path shown here (e.g., `webhook/1234-abcd...`) is the unique URL for your API endpoint.

    Your configuration should look like this:
    ![Webhook Settings](https://i.imgur.com/L6gR7hP.png)
    *(Note: Image is for illustrative purposes)*

4.  Copy the **Test URL**. You will use this to send test data to activate the node.

### Part 2: The Compression Logic (Code Node)

This node will contain the JavaScript code to perform the compression.

1.  Add a **Code** node and connect it to the Webhook node.
2.  Paste the following JavaScript code into the Code node's editor. This example uses the `sharp` library, which is a common choice for this task.

    ```javascript
    // This example uses the 'sharp' library for image processing.
    // You may need to make it available in your n8n instance.
    const sharp = require('sharp');

    // Get the incoming image data from the Webhook node.
    // n8n makes the binary data available in the '$binary' property.
    const incomingImageBuffer = await $binary.data.toBuffer();

    // --- YOUR COMPRESSION LOGIC GOES HERE ---
    // This is an example using sharp to convert to JPEG with quality 75.
    // You should replace this with your own compression logic.
    const compressedImageBuffer = await sharp(incomingImageBuffer)
      .jpeg({ quality: 75 })
      .toBuffer();
    // -----------------------------------------

    // Return the compressed image as a new binary object.
    // n8n requires a specific structure for binary output.
    // We give it a new name, e.g., 'compressedImage.jpg'.
    const binaryData = await $helpers.prepareBinaryData(compressedImageBuffer, 'compressedImage.jpg', 'image/jpeg');

    // Return the final item so the webhook can respond with the image.
    return {
      binary: binaryData,
    };
    ```

3.  **Important:** You must replace the example `sharp` logic with your own compression algorithm from your `Compress` project, adapted for Node.js.

### Part 3: Responding to the API Call

There is no Step 3! When a workflow is triggered by a Webhook, and a node connected to it returns a single item with a `binary` property, the Webhook will **automatically** respond to the HTTP request with the binary data (the file) and the correct `Content-Type` header (`image/jpeg` in our example).

Your final workflow should look very simple:
**Webhook -> Code**

---

## How to Use Your New API

You can now call this API endpoint from any application or using a command-line tool like `curl`.

To test it, save the workflow and use the **Production URL** from the Webhook node.

```bash
# Replace with your image file and your n8n production URL
curl -X POST \
     --data-binary @"/path/to/your/image.png" \
     -H "Content-Type: image/png" \
     "YOUR_N8N_PRODUCTION_WEBHOOK_URL" \
     --output "compressed_output.jpg"

echo "Image compressed and saved to compressed_output.jpg"
```

You have now successfully turned your compression tool into a scalable, on-demand API service using n8n.
