## Websocket

This document explains how to implement image uploading functionality in the chat system using WebSocket connections. The backend supports both private and group chat image messages.

## Message Format

### Private Chat Image Message

```javascript
{
    "type": "privateChat",
    "chat_id": "unique_chat_id",
    "receiver_username": "recipient_username",
    "message_text": "Optional caption for the image",
    "image_data": "base64_encoded_image_data"
}
```

### Group Chat Image Message

```javascript
{
    "type": "groupChat",
    "group_id": "unique_group_id",
    "message_text": "Optional caption for the image",
    "image_data": "base64_encoded_image_data"
}
```

## Implementation Steps

1. **Image Selection**

   ```javascript
   // Example using input type file
   const fileInput = document.querySelector('input[type="file"]');
   fileInput.addEventListener('change', handleImageSelect);

   function handleImageSelect(event) {
     const file = event.target.files[0];
     if (!file) return;

     // Validate file type
     if (!file.type.startsWith('image/')) {
       alert('Please select an image file');
       return;
     }

     // Convert to base64
     const reader = new FileReader();
     reader.onload = (e) => {
       const base64Data = e.target.result;
       // Send the image
       sendImageMessage(base64Data);
     };
     reader.readAsDataURL(file);
   }
   ```
2. **Sending Image Messages**

   ```javascript
   function sendPrivateImageMessage(base64Data, chatId, receiverUsername, caption = '') {
     const message = {
       type: 'privateChat',
       chat_id: chatId,
       receiver_username: receiverUsername,
       message_text: caption,
       image_data: base64Data
     };

     websocket.send(JSON.stringify(message));
   }

   function sendGroupImageMessage(base64Data, groupId, caption = '') {
     const message = {
       type: 'groupChat',
       group_id: groupId,
       message_text: caption,
       image_data: base64Data
     };

     websocket.send(JSON.stringify(message));
   }
   ```
3. **Handling Image Messages**

   ```javascript
   websocket.onmessage = function(event) {
     const data = JSON.parse(event.data);

     if (data.type === 'privateChat' || data.type === 'groupChat') {
       if (data.image_url) {
         // Display the image
         displayChatImage(data);
       }
     }
   };

   function displayChatImage(messageData) {
     const imageContainer = document.createElement('div');
     imageContainer.className = 'chat-image-container';

     const img = document.createElement('img');
     img.src = messageData.image_url;
     img.alt = 'Chat Image';

     if (messageData.message_text) {
       const caption = document.createElement('p');
       caption.textContent = messageData.message_text;
       imageContainer.appendChild(caption);
     }

     imageContainer.appendChild(img);
     // Add to your chat container
     chatContainer.appendChild(imageContainer);
   }
   ```

## Response Format

When an image is successfully sent, you'll receive a message with the following format:

```javascript
{
    "type": "privateChat" | "groupChat",
    "id": "chat_id" | "group_id",
    "slug": "chat_slug",
    "sender": {
        "id": "user_id",
        "first_name": "First",
        "last_name": "Last",
        "username": "username",
        "profile_url": "profile_picture_url"
    },
    "message_text": "Optional caption",
    "image_url": "https://assets.socialverseapp.com/path/to/image.jpg",
    "sent_at": "2024-02-20 12:34:56"
}
```

## Error Handling

```javascript
websocket.onmessage = function(event) {
  const data = JSON.parse(event.data);
  
  if (data.type === 'error') {
    console.error('Chat error:', data.message);
    // Handle error appropriately in your UI
    showErrorMessage(data.message);
  }
};
```

## Best Practices

1. **Image Validation**

   - Validate file type before upload
   - Check file size (recommend < 5MB)
   - Support common image formats (JPEG, PNG, GIF)
2. **UI Considerations**

   - Show loading indicator while image is uploading
   - Provide preview before sending
   - Allow caption input
   - Show error messages if upload fails
3. **Performance**

   - Compress images client-side if possible
   - Cache received images
   - Implement lazy loading for chat history

## Example Implementation

```javascript
class ChatImageHandler {
  constructor(websocket) {
    this.websocket = websocket;
    this.maxSizeBytes = 5 * 1024 * 1024; // 5MB
  }

  async handleImageUpload(file, chatType, id, receiverUsername = null) {
    try {
      // Validate file
      if (!this.validateImage(file)) return;
  
      // Convert to base64
      const base64Data = await this.fileToBase64(file);
  
      // Show preview
      this.showImagePreview(base64Data);
  
      // Get caption (implement based on your UI)
      const caption = await this.getCaptionFromUser();
  
      // Send based on chat type
      if (chatType === 'private') {
        this.sendPrivateImageMessage(base64Data, id, receiverUsername, caption);
      } else {
        this.sendGroupImageMessage(base64Data, id, caption);
      }
    } catch (error) {
      console.error('Image upload failed:', error);
      this.showErrorMessage('Failed to upload image. Please try again.');
    }
  }

  validateImage(file) {
    if (!file.type.startsWith('image/')) {
      this.showErrorMessage('Please select an image file.');
      return false;
    }
  
    if (file.size > this.maxSizeBytes) {
      this.showErrorMessage('Image size should be less than 5MB.');
      return false;
    }
  
    return true;
  }

  fileToBase64(file) {
    return new Promise((resolve, reject) => {
      const reader = new FileReader();
      reader.onload = (e) => resolve(e.target.result);
      reader.onerror = (e) => reject(e);
      reader.readAsDataURL(file);
    });
  }
}
```

## Notes

- The backend automatically optimizes images and converts them to JPEG format
- Images are stored in the `socialverse-assets` S3 bucket
- Image URLs follow the pattern: `https://assets.socialverseapp.com/path/to/image.jpg`
- Maximum recommended image size is 5MB
- Supported image formats: JPEG, PNG, GIF
