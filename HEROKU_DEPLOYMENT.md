# Deploying the Company Research Agent to Heroku

This guide provides instructions for deploying the Company Research Agent backend to Heroku.

## Prerequisites

- A Heroku account
- Heroku CLI installed on your computer
- Git installed on your computer
- Required API keys:
  - OpenAI API key
  - Tavily API key
  - Google Gemini API key (optional)
  - MongoDB URI (optional for persistence)

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
heroku config:set OPENAI_API_KEY=your_openai_api_key
heroku config:set TAVILY_API_KEY=your_tavily_api_key
heroku config:set GEMINI_API_KEY=your_gemini_api_key
```

If you're using MongoDB for persistence:

```bash
heroku config:set MONGODB_URI=your_mongodb_uri
```

### 4. Deploy using Git

```bash
git push heroku main
```

### 5. Deploy using Docker (Alternative)

If you prefer using Docker deployment with Heroku:

```bash
heroku container:login
heroku container:push web
heroku container:release web
```

### 6. Verify the deployment

```bash
heroku open
```

## Connecting the Frontend

To connect the frontend to your Heroku backend, update the following environment variables in your frontend:

```
VITE_API_URL=https://your-app-name.herokuapp.com
VITE_WS_URL=wss://your-app-name.herokuapp.com
```

## Troubleshooting

- If the app fails to start, check the logs:
  ```bash
  heroku logs --tail
  ```

- Make sure all required environment variables are set properly:
  ```bash
  heroku config
  ```

- You may need to scale up the dyno to handle the application:
  ```bash
  heroku ps:scale web=1
  ```

## Additional Resources

- [Heroku Python Support](https://devcenter.heroku.com/articles/python-support)
- [Heroku Docker Deployment](https://devcenter.heroku.com/articles/container-registry-and-runtime)
- [Managing Heroku Environment Variables](https://devcenter.heroku.com/articles/config-vars) 