# Deploying the Company Research Agent to Heroku

This guide provides comprehensive instructions for deploying the Company Research Agent backend to Heroku, including detailed troubleshooting steps.

## Prerequisites

- A Heroku account
- Heroku CLI installed on your computer
- Git installed on your computer
- Required API keys:
  - OpenAI API key
  - Tavily API key
  - Google Gemini API key (optional)
  - MongoDB URI (optional for persistence)

## Required Files

Ensure these files are in your repository root:

1. `Procfile` - Tells Heroku how to run your application:
   ```
   web: uvicorn application:app --host=0.0.0.0 --port=$PORT
   ```

2. `runtime.txt` - Specifies the Python version:
   ```
   python-3.11.7
   ```

3. `requirements.txt` - Lists all dependencies

## Deployment Steps

### 1. Login to Heroku

```bash
heroku login
```

### 2. Create a new Heroku app

```bash
heroku create your-app-name
```

### 3. Set up Heroku environment variables

```bash
heroku config:set OPENAI_API_KEY=your_openai_api_key --app your-app-name
heroku config:set TAVILY_API_KEY=your_tavily_api_key --app your-app-name
heroku config:set GEMINI_API_KEY=your_gemini_api_key --app your-app-name
```

If you're using MongoDB for persistence, make sure to include SSL parameters to prevent connection issues:

```bash
heroku config:set MONGODB_URI="mongodb+srv://username:password@cluster.mongodb.net/?retryWrites=true&w=majority&tls=true&tlsAllowInvalidCertificates=true&appName=YourAppName" --app your-app-name
```

### 4. Deploy using Git

First, commit your changes:

```bash
git add .
git commit -m "Ready for Heroku deployment"
```

If you're on a feature branch and want to deploy it:

```bash
git push heroku your-feature-branch:main
```

For the main branch:

```bash
git push heroku main
```

### 5. Scale up your dynos

After deployment, you must scale up the web dyno to start serving requests:

```bash
heroku ps:scale web=1 --app your-app-name
```

### 6. Verify the deployment

Check if the application is running:

```bash
heroku ps --app your-app-name
```

Test the API:

```bash
curl https://your-app-name.herokuapp.com/
```

You should receive a response like: `{"message":"Alive"}`

## Connecting the Frontend

To connect the frontend to your Heroku backend, update the following environment variables in your frontend:

```
VITE_API_URL=https://your-app-name.herokuapp.com
VITE_WS_URL=wss://your-app-name.herokuapp.com
```

## Troubleshooting Common Issues

### 1. "No web processes running" (H14 Error)

**Symptom**: You see "at=error code=H14 desc='No web processes running'" in logs.

**Solution**: Scale up your web dyno:
```bash
heroku ps:scale web=1 --app your-app-name
```

### 2. Port Configuration Issues

**Symptom**: App crashes or fails to bind to port.

**Solution**: Ensure your Procfile uses the `$PORT` environment variable:
```
web: uvicorn application:app --host=0.0.0.0 --port=$PORT
```

And your application code uses:
```python
port = int(os.getenv("PORT", 8000))
```

### 3. MongoDB Connection Issues

**Symptom**: SSL handshake errors when connecting to MongoDB Atlas:
```
SSL handshake failed: [SSL: TLSV1_ALERT_INTERNAL_ERROR] tlsv1 alert internal error
```

**Cause**: This is primarily due to Network Access limitations in MongoDB Atlas. By default, MongoDB only allows connections from whitelisted IP addresses.

**Solution**: 

1. Configure MongoDB Atlas Network Access:
   - Log in to your MongoDB Atlas account
   - Navigate to Network Access under Security in the left sidebar
   - Click "Add IP Address"
   - Add `0.0.0.0/0` to allow connections from anywhere (for testing)
   - Or add specific Heroku IP ranges for better security
   - Click "Save & Confirm"
   - Wait for changes to propagate (approximately 5 minutes)

2. Update your MongoDB connection string with proper SSL parameters:
```bash
heroku config:set MONGODB_URI="mongodb+srv://username:password@cluster.mongodb.net/?retryWrites=true&w=majority&tls=true&tlsAllowInvalidCertificates=true&appName=YourAppName" --app your-app-name
```

3. Restart your application to apply the changes:
```bash
heroku restart --app your-app-name
```

> **Note**: Using `0.0.0.0/0` (allow access from anywhere) is recommended only for testing. For production, consider using more restrictive IP ranges or implementing additional security measures like VPC peering.

### 4. WebSocket Connection Issues

**Symptom**: WebSocket connections not working with wss:// protocol.

**Solution**: 
- Ensure WebSockets are correctly configured in application.py
- Confirm the client is using wss:// protocol (not ws://) for secure connections
- Check if there are any proxy settings interfering with WebSocket connections

### 5. Application Crashes or Timeouts

**Symptom**: Application crashes shortly after starting or times out on requests.

**Solution**: Check your application logs:
```bash
heroku logs --tail --app your-app-name
```

You might need to increase dyno resources or optimize your application for better performance.

## Monitoring and Maintenance

### Checking Application Status

```bash
heroku ps --app your-app-name
```

### Viewing Application Logs

```bash
heroku logs --tail --app your-app-name
```

Filter logs for specific issues:
```bash
heroku logs --app your-app-name | grep "MongoDB"
heroku logs --app your-app-name | grep "WebSocket"
heroku logs --app your-app-name | grep "ERROR"
```

### Restarting the Application

If the application is behaving unexpectedly after configuration changes:

```bash
heroku restart --app your-app-name
```

## Additional Resources

- [Heroku Python Support](https://devcenter.heroku.com/articles/python-support)
- [Heroku Docker Deployment](https://devcenter.heroku.com/articles/container-registry-and-runtime)
- [Managing Heroku Environment Variables](https://devcenter.heroku.com/articles/config-vars)
- [WebSockets on Heroku](https://devcenter.heroku.com/articles/websockets)
- [Troubleshooting Memory Issues](https://devcenter.heroku.com/articles/troubleshooting-memory-issues)
- [Heroku Error Codes](https://devcenter.heroku.com/articles/error-codes) 