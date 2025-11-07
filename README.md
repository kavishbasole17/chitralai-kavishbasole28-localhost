# Chitralai - AI-Powered Image Search Application

An intelligent image search application that automatically analyzes uploaded images using AWS Rekognition and allows users to search for images using natural language keywords.

## 🎯 Overview

**Chitralai** is a full-stack web application featuring:

- **Frontend:** React-based UI with drag-and-drop image upload and grid-based search results
- **Backend:** Express.js server with presigned S3 URLs and DynamoDB search capabilities
- **AI Analysis:** AWS Rekognition for automatic image labeling
- **Serverless:** AWS Lambda for event-driven image processing
- **Storage:** AWS S3 for images, DynamoDB for metadata

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     React Frontend                           │
│        (Upload Component + Search Bar + Image Grid)          │
└──────────────────────────┬──────────────────────────────────┘
                           │ API Calls
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                  Express.js Backend                          │
│   (Presigned URL Generation + DynamoDB Search)              │
└──────────────┬──────────────────────────────┬────────────────┘
               │                              │
    PUT Presigned URL              Query/Scan
               │                              │
               ▼                              ▼
        ┌────────────┐              ┌────────────────┐
        │  AWS S3    │              │  DynamoDB      │
        │  (Images)  │              │  (Metadata)    │
        └─────┬──────┘              └────────────────┘
              │
              │ S3 Event
              ▼
        ┌────────────────┐
        │  AWS Lambda    │
        │ (Rekognition)  │
        └─────┬──────────┘
              │
              └─────► Rekognition (Detect Labels)
              │
              └─────► DynamoDB (Store Tags)
```

## 📦 Project Structure

```
chitralai/
├── frontend/                 # React application
│   ├── src/
│   │   ├── components/      # React components
│   │   │   ├── ImageUpload.js
│   │   │   ├── SearchBar.js
│   │   │   └── ImageGrid.js
│   │   ├── utils/
│   │   │   └── api.js       # API client
│   │   ├── styles/
│   │   │   └── App.css
│   │   ├── App.js
│   │   └── index.js
│   ├── index.html
│   ├── vite.config.js
│   ├── package.json
│   └── .env.local
│
├── backend/                  # Express server
│   ├── src/
│   │   ├── controllers/     # Route handlers
│   │   │   ├── uploadController.js
│   │   │   └── searchController.js
│   │   ├── services/        # Business logic
│   │   │   ├── s3Service.js
│   │   │   └── dynamoService.js
│   │   ├── middleware/
│   │   │   └── errorHandler.js
│   │   └── server.js
│   ├── package.json
│   └── .env
│
├── lambda/                   # AWS Lambda function
│   ├── index.js
│   ├── package.json
│   └── .env
│
└── aws-setup/               # AWS configuration
    ├── setup-instructions.md
    └── iam-policy.json
```

## 🚀 Quick Start

### Prerequisites

- Node.js 18.x or later
- AWS Account with appropriate permissions
- Git

### 1. Clone Repository

```bash
git clone <repository-url>
cd chitralai
```

### 2. Setup Frontend

```bash
cd frontend
npm install
npm run dev
```

Frontend will run on `http://localhost:3000`

### 3. Setup Backend

```bash
cd ../backend
npm install
npm run dev
```

Backend will run on `http://localhost:5000`

### 4. AWS Configuration

Follow the comprehensive AWS setup guide:

```bash
cat aws-setup/setup-instructions.md
```

Key steps:
1. Create S3 bucket
2. Create DynamoDB table
3. Create IAM roles and user
4. Deploy Lambda function
5. Configure S3 event notification
6. Update environment variables

## 📝 API Endpoints

### POST /api/generate-upload-url

Generate a presigned URL for uploading an image directly to S3.

**Request:**
```json
{
  "fileName": "photo.jpg",
  "fileType": "image/jpeg"
}
```

**Response:**
```json
{
  "presignedUrl": "https://bucket.s3.amazonaws.com/...",
  "imageId": "uuid-string",
  "expiresIn": 900
}
```

### GET /api/search

Search for images by keywords.

**Request:**
```
GET /api/search?q=beach&q=sunset&q=water
```

**Response:**
```json
{
  "count": 3,
  "images": [
    {
      "id": "image-uuid",
      "url": "https://bucket.s3.amazonaws.com/uploads/image.jpg",
      "tags": ["beach", "sunset", "water", "sand"],
      "uploadedAt": "2024-01-15T10:30:00Z"
    }
  ]
}
```

## 🔄 Data Flow

### Upload Flow

1. User selects image in frontend
2. Frontend validates file (type, size)
3. Frontend calls `POST /api/generate-upload-url`
4. Backend generates presigned URL and returns it
5. Frontend performs `PUT` directly to S3 using presigned URL
6. Frontend displays success message

### Analysis Flow

1. S3 triggers Lambda when image is uploaded
2. Lambda calls Rekognition `detectLabels` API
3. Rekognition returns labels with confidence scores
4. Lambda filters labels (>80% confidence)
5. Lambda stores metadata in DynamoDB:
   - imageId
   - s3Key
   - s3Url
   - tags (array of label names, lowercase)
   - uploadTimestamp

### Search Flow

1. User types keywords and submits search
2. Frontend calls `GET /api/search?q=keyword1&q=keyword2`
3. Backend queries DynamoDB Scan with FilterExpression
4. All keywords must exist in image tags
5. Backend returns matching images with URLs
6. Frontend displays results in grid

## 🔑 Environment Variables

### Frontend (.env.local)

```
VITE_API_BASE_URL=http://localhost:5000
```

### Backend (.env)

```
PORT=5000
NODE_ENV=development
FRONTEND_URL=http://localhost:3000

AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=your-key
AWS_SECRET_ACCESS_KEY=your-secret
S3_BUCKET_NAME=my-bucket
DYNAMODB_TABLE_NAME=images-metadata
```

### Lambda (.env)

```
AWS_REGION=us-east-1
DYNAMODB_TABLE_NAME=images-metadata
REKOGNITION_MIN_CONFIDENCE=80
```

## 💾 Database Schema

### DynamoDB Table: images-metadata

| Attribute | Type | Description |
|-----------|------|-------------|
| imageId | String (PK) | Unique image identifier |
| s3Key | String | S3 object key (uploads/...) |
| s3Url | String | Public S3 URL |
| tags | List<String> | Detected labels (lowercase) |
| uploadTimestamp | String | ISO 8601 timestamp |

## 🔐 Security Considerations

- **Presigned URLs:** Time-limited (15 min), single-use URLs for secure uploads
- **CORS:** Restricted to frontend origin
- **IAM Policies:** Principle of least privilege for Lambda and backend
- **S3 Bucket:** Block all public access by default
- **Environment Variables:** Store secrets in .env files (not in code)

## ⚙️ Configuration

### Image Upload

- **Accepted formats:** JPEG, PNG, GIF, WebP
- **Max file size:** 10MB
- **Presigned URL expiration:** 15 minutes

### Search

- **Min keywords:** 1
- **Max keywords:** 5
- **Max results:** 100 images
- **Matching logic:** ALL keywords must be present in image tags

### Rekognition

- **Min confidence:** 80%
- **Max labels:** 100
- **Max tags stored:** 20 per image

## 🧪 Testing

### Manual Testing

1. **Upload Flow:**
   - Upload an image through the UI
   - Verify it appears in S3 bucket
   - Check Lambda CloudWatch logs
   - Verify tags in DynamoDB

2. **Search Flow:**
   - Search for detected tags
   - Verify images appear in grid
   - Try multiple keywords

3. **Error Handling:**
   - Try uploading non-image file
   - Upload file >10MB
   - Search with >5 keywords
   - Test network errors

## 📊 Monitoring

### CloudWatch Logs

- **Backend:** Application logs (npm run dev)
- **Lambda:** `/aws/lambda/image-analysis-lambda`
- **S3 Events:** Check Lambda CloudWatch logs

## 🐛 Troubleshooting

See `aws-setup/setup-instructions.md` for detailed troubleshooting guide.

Common issues:
- **S3 CORS errors:** Configure CORS policy
- **Lambda not triggered:** Check S3 event notification
- **No tags detected:** Verify Rekognition permissions and image quality
- **Search returns empty:** Ensure keywords match detected tags

## 📈 Performance Tips

1. **Frontend:**
   - Optimize images before upload
   - Use progressive image loading

2. **Backend:**
   - Add caching for search results
   - Implement pagination for large result sets

3. **Lambda:**
   - Increase memory allocation for faster Rekognition calls

4. **DynamoDB:**
   - Switch to provisioned capacity for production

## 🚢 Production Deployment

### Frontend
```bash
npm run build
# Deploy dist/ to Vercel, Netlify, or S3+CloudFront
```

### Backend
- Deploy to Heroku, Railway, or AWS services
- Update environment variables

### Lambda
```bash
zip -r lambda-function.zip lambda/
aws lambda update-function-code --function-name image-analysis-lambda --zip-file fileb://lambda-function.zip
```

## 📄 License

This project is provided as-is for educational and commercial use.

## 📞 Support

For issues and questions:
1. Check the troubleshooting guide
2. Review AWS setup instructions
3. Check application logs
4. Open an issue in the repository

---

**Built with React, Express.js, and AWS services** 🚀#   c h i t r a l a i - f i n a l  
 