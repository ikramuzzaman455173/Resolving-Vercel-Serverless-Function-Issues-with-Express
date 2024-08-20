# Resolving Vercel Serverless Function Issues with Express

## Table of Contents
1. [Introduction](#introduction)
2. [Problem Description](#problem-description)
3. [Solution Overview](#solution-overview)
4. [Step-by-Step Solution](#step-by-step-solution)
   - [1. Modify `api/index.js`](#1-modify-apindexjs)
   - [2. Update `vercel.json`](#2-update-verceljson)
   - [3. Test Locally](#3-test-locally)
   - [4. Deploy to Vercel](#4-deploy-to-vercel)
5. [Full Code Example](#full-code-example)
6. [Go to Top](#go-to-top)

---

## Introduction

This documentation provides a comprehensive guide to resolving issues with deploying an Express application as a serverless function on Vercel. The common issue faced is the `TypeError: listener is not a function`, which often arises due to incorrect setup for serverless functions.

## Problem Description

When deploying an Express application to Vercel, you might encounter the error:

```
TypeError: listener is not a function
```

This error indicates that Vercel's serverless environment is not handling the Express app correctly due to improper setup.

## Solution Overview

To resolve the issue, follow these steps:
1. **Modify `api/index.js`**: Adjust the serverless function to export a handler compatible with Vercel.
2. **Update `vercel.json`**: Ensure the configuration points to the correct file and setup.
3. **Test Locally**: Verify the setup using local testing.
4. **Deploy to Vercel**: Deploy your application and ensure it works correctly on Vercel.

## Step-by-Step Solution

### 1. Modify `api/index.js`

Update your `api/index.js` file to export a handler function that Vercel can use. Here’s the revised code:

```javascript
import express from 'express';
import dotenv from 'dotenv';
import cookieParser from 'cookie-parser';
import fileUpload from 'express-fileupload';
import cors from 'cors';
import mongoose from 'mongoose';
import morgan from 'morgan';
import router from '../routes/allRoutes.js';

// Load environment variables from .env file
dotenv.config();

// Initialize Express app
const app = express();

// Setup CORS
app.use(
  cors({
    origin: ['http://localhost:5173', 'http://localhost:3001'],
    credentials: true,
  })
);

// Middleware setup
app.use(cookieParser());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Setup basic Morgan logger
app.use(morgan('dev'));

// File upload configuration
app.use(
  fileUpload({
    useTempFiles: true,
    tempFileDir: '/tmp/',
  })
);

// MongoDB connection
const MongoUrl = process.env.MONGO_URI;

mongoose
  .connect(MongoUrl)
  .then(() => {
    console.log('Connected to MongoDB successfully!');
  })
  .catch((err) => {
    console.error(`Failed to connect to MongoDB: ${err.message}`);
  });

// Setup API routes (add your routes here)
app.use('/api/v1', router);

// Root path with a welcome message and list of all GET routes for fetching data
app.get('/', (req, res) => {
  res.json({
    message: 'Welcome to the IJ portfolio backend API!',
    status: 200,
    'GET Routes': {
      'Get User': '/api/v1/user/me',
      'Get User for Portfolio': '/api/v1/user/portfolio/me',
      'Get All Timelines': '/api/v1/timeline/getAllTimeline',
      'Get All Software Applications': '/api/v1/softwareApplication/getAllSoftwareApplication',
      'Get All Skills': '/api/v1/skill/getAllSkill',
      'Get Grouped Skills': '/api/v1/skill/getGroupedAllSkills',
      'Get All Projects': '/api/v1/project/getAllProject',
      'Get All Messages': '/api/v1/message/getAllMessages',
      'Get All Certificates': '/api/v1/certificate/getAllCertificates',
      'Get All Blogs': '/api/v1/blog/getAllBlogs',
    },
  });
});

// Error handling middleware
app.use((err, req, res, next) => {
  console.error('Error:', err.stack);
  res.status(500).send('Something broke!');
});

// Export the handler for Vercel
export default (req, res) => {
  return app(req, res);
};
```

### 2. Update `vercel.json`

Ensure your `vercel.json` file is configured to point to the correct function:

```json
{
  "version": 2,
  "builds": [
    {
      "src": "api/index.js",
      "use": "@vercel/node"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "/api/index.js",
      "methods": ["GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"]
    }
  ]
}
```

### 3. Test Locally

Test your setup locally using the Vercel CLI:

```bash
vercel dev
```

Ensure that your Express app runs as expected on `http://localhost:3000`.

### 4. Deploy to Vercel

Deploy your application to Vercel:

```bash
vercel --prod
```

Ensure that the deployment succeeds and the application works correctly on Vercel.

## Full Code Example

```javascript
// Full `api/index.js` code provided in the "Modify `api/index.js`" section
```

## Go to Top

[Go to Top](#resolving-vercel-serverless-function-issues-with-express)

---
