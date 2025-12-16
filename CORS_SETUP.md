# CORS Configuration Guide

## Problem
The frontend on Vercel fails to create sessions because the backend CORS configuration only allowed a single origin.

## Solution
The backend now supports multiple CORS origins through comma-separated CLIENT_URL values.

## Setup Instructions

### For Local Development Only
```env
CLIENT_URL=http://localhost:5173
```

### For Production (Vercel) + Local Development
```env
CLIENT_URL=http://localhost:5173,https://your-app.vercel.app
```

### For Multiple Production Domains
```env
CLIENT_URL=http://localhost:5173,https://your-app.vercel.app,https://your-app-staging.vercel.app
```

## How It Works

1. **Environment Variable Parsing**: The `CLIENT_URL` environment variable is split by commas and trimmed to create an array of allowed origins.

2. **CORS Validation**: When a request comes in, the CORS middleware checks if the request origin is in the allowed list.

3. **Special Cases**:
   - Requests without an origin (e.g., from mobile apps, Postman) are automatically allowed
   - If `CLIENT_URL` is empty or not set, requests are allowed (development mode)

## Deployment Steps

1. **Update your environment variables** on your hosting platform (Render, Heroku, AWS, etc.):
   ```
   CLIENT_URL=http://localhost:5173,https://your-vercel-app.vercel.app
   ```

2. **Redeploy** your backend service

3. **Verify** the CORS configuration by checking server logs for "CORS blocked" messages

## Troubleshooting

### CORS Still Blocking Requests?

1. Check server logs for "CORS blocked request from origin: <URL>"
2. Verify the exact origin URL matches what's in CLIENT_URL (including protocol and port)
3. Ensure no trailing slashes in URLs
4. Check that CLIENT_URL is properly set in your production environment

### Common Issues

- **Wrong protocol**: Make sure to use `https://` for production Vercel URLs, not `http://`
- **Missing www**: If your domain uses `www`, include it: `https://www.example.com`
- **Port numbers**: For local development, include the port: `http://localhost:5173`
- **Preview deployments**: Vercel creates unique URLs for each preview deployment. You may need to add them or use a wildcard pattern (requires code changes)

## Security Notes

- Only add trusted origins to CLIENT_URL
- Do not use wildcards (*) in production
- Review and update allowed origins regularly
- Monitor CORS blocked requests in logs

## Example Configuration

### Local Development
```bash
# .env
CLIENT_URL=http://localhost:5173
```

### Production (Vercel Frontend + Render Backend)
```bash
# Environment variables on Render
CLIENT_URL=https://your-app.vercel.app,https://your-app-staging.vercel.app
```

### Production + Local Testing
```bash
# Environment variables on Render
CLIENT_URL=http://localhost:5173,https://your-app.vercel.app
```

## Testing the Configuration

You can test CORS by:

1. **Browser Console**: Check for CORS errors in the browser console
2. **Network Tab**: Look for OPTIONS (preflight) requests
3. **Server Logs**: Monitor for "CORS blocked" messages
4. **curl**: Test from command line:
   ```bash
   curl -H "Origin: https://your-app.vercel.app" \
        -H "Access-Control-Request-Method: POST" \
        -H "Access-Control-Request-Headers: Content-Type" \
        -X OPTIONS \
        https://your-backend.render.com/api/sessions
   ```

   Expected response should include:
   ```
   Access-Control-Allow-Origin: https://your-app.vercel.app
   Access-Control-Allow-Credentials: true
   ```
