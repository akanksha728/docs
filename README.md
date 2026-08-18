Today 11:13 AM

Add
Request failed with status code 502 index-DcgBRhtA.js:13
GET https://droneyaan.com/cameras/ 502 (Bad Gateway)
(anonymous) @ index-DcgBRhtA.js:13
xhr @ index-DcgBRhtA.js:13
po @ index-DcgBRhtA.js:15
Promise.then
_request @ index-DcgBRhtA.js:19
request @ index-DcgBRhtA.js:15
B.<computed> @ index-DcgBRhtA.js:19
(anonymous) @ index-DcgBRhtA.js:11
Ko @ index-DcgBRhtA.js:19
(anonymous) @ index-DcgBRhtA.js:19
index-DcgBRhtA.js:13
GET https://droneyaan.com/cameras/ 502 (Bad Gateway)
index-DcgBRhtA.js:19
WebSocket connection to 'wss://droneyaan.com/ws/ai' failed: Error during WebSocket handshake: Unexpected response code: 502
index-DcgBRhtA.js:13
GET https://droneyaan.com/cameras/ 502 (Bad Gateway)
index-DcgBRhtA.js:19
WebSocket connection to 'wss://droneyaan.com/ws/ai' failed: Error during WebSocket handshake: Unexpected response code: 502
index-DcgBRhtA.js:13
GET https://droneyaan.com/cameras/ 502 (Bad Gateway)
index-DcgBRhtA.js:13
GET https://droneyaan.com/cameras/ 502 (Bad Gateway)
index-DcgBRhtA.js:19
WebSocket connection to 'wss://droneyaan.com/ws/ai' failed: Error during WebSocket handshake: Unexpected response code: 502
index-DcgBRhtA.js:13
GET https://droneyaan.com/cameras/ 502 (Bad Gateway)
index-DcgBRhtA.js:13
GET https://droneyaan.com/cameras/ 502 (Bad Gateway)
index-DcgBRhtA.js:13
GET https://droneyaan.com/cameras/ 502 (Bad Gateway)
index-DcgBRhtA.js:19
WebSocket connection to 'wss://droneyaan.com/ws/ai' failed: Error during WebSocket handshake: Unexpected response code: 502
index-DcgBRhtA.js:13
GET https://droneyaan.com/cameras/ 502 (Bad Gateway)
index-DcgBRhtA.js:19
WebSocket connection to 'wss://droneyaan.com/ws/ai' failed: Error during WebSocket handshake: Unexpected response code: 502
index-DcgBRhtA.js:13
GET https://droneyaan.com/cameras/ 502 (Bad Gateway)
﻿

given these issus

These errors are actually helpful: your frontend is reaching https://droneyaan.com, but Nginx cannot reach the FastAPI backend correctly.

You have two related production routing problems:

GET https://droneyaan.com/cameras/       → 502
WSS https://droneyaan.com/ws/ai          → 502

The most likely issue is your Nginx reverse-proxy configuration / FastAPI upstream, not React.

1. First verify FastAPI on AWS

SSH into your EC2 and run:

sudo ss -lntp | grep 8000

You should see something like:

127.0.0.1:8000

Then:

curl -i http://127.0.0.1:8000/cameras/
If you get JSON

For example:

[]

or a camera list, then FastAPI is running and the problem is Nginx.

If you get:
Connection refused

then FastAPI isn't running/listening on port 8000.

Run:

ps aux | grep uvicorn

and:

sudo systemctl status indoplayer

If you don't have a systemd service yet, tell me what command you're currently using to start FastAPI.

2. Check your FastAPI route

Your camera router from earlier is:

@router.get("/")
def get_cameras():
    ...

We need to know how it is included in app/main.py.

Look for something like:

app.include_router(camera_router, prefix="/cameras")

If that's the case, your backend route is:

http://127.0.0.1:8000/cameras/

That's exactly what the frontend is requesting:

https://droneyaan.com/cameras/

So Nginx should proxy /cameras/ to FastAPI.

3. Fix Nginx API routing

Your Nginx server should have a configuration similar to this:

server {
    listen 80;
    server_name droneyaan.com www.droneyaan.com;


    root /var/www/html;
    index index.html;


    # React
    location / {
        try_files $uri $uri/ /index.html;
    }


    # FastAPI REST API
    location /cameras/ {
        proxy_pass http://127.0.0.1:8000/cameras/;


        proxy_http_version 1.1;


        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;


        proxy_read_timeout 3600s;
        proxy_send_timeout 3600s;
    }


    # AI WebSocket
    location /ws/ai {
        proxy_pass http://127.0.0.1:8000/ws/ai;


        proxy_http_version 1.1;


        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";


        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;


        proxy_read_timeout 3600s;
        proxy_send_timeout 3600s;
    }


    # Publisher WebSocket
    location /api/publisher/ws/ {
        proxy_pass http://127.0.0.1:8000/api/publisher/ws/;


        proxy_http_version 1.1;


        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";


        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;


        proxy_read_timeout 3600s;
        proxy_send_timeout 3600s;
    }
}

But don't blindly replace your current Nginx file yet. Your exact FastAPI prefixes need to match your actual main.py.

4. There may be more API routes

Your frontend probably calls more than:

/cameras/

You have things such as:

/alerts
/alerts/unread
/recordings
/analytics
/auth
/ws/ai
/api/publisher/ws/...

If your existing Nginx has a general API proxy, it may be much cleaner to use:

location / {
    try_files $uri $uri/ /index.html;
}


location /api/ {
    proxy_pass http://127.0.0.1:8000;


    proxy_http_version 1.1;


    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}

However, your current frontend is requesting /cameras/, not /api/cameras/.

Therefore, either:

Option A — Keep your current frontend URLs

Proxy:

/cameras/

directly to:

127.0.0.1:8000/cameras/
Option B — Move production API URLs to /api/...

Frontend:

https://droneyaan.com/api/cameras/

Nginx:

/api/ → FastAPI

This is generally cleaner, but it requires checking your existing frontend/backend routes before changing it.

For your production project, I recommend Option A for now because you want to avoid breaking the already-working application.

5. Test Nginx configuration

After modifying Nginx:

sudo nginx -t

You must get:

syntax is ok
test is successful

Then:

sudo systemctl reload nginx
6. Test through the domain

On the EC2:

curl -i https://droneyaan.com/cameras/

You want:

HTTP/2 200

with your camera JSON.

If you still get:

502 Bad Gateway

immediately run:

sudo tail -50 /var/log/nginx/error.log

That log will tell us exactly why Nginx is returning 502.

7. Fix /ws/ai

Your error:

WebSocket connection to
'wss://droneyaan.com/ws/ai'
failed


Unexpected response code: 502

requires WebSocket proxying.

Nginx must have:

location /ws/ai {
    proxy_pass http://127.0.0.1:8000/ws/ai;


    proxy_http_version 1.1;


    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";


    proxy_set_header Host $host;


    proxy_read_timeout 3600s;
}

The important parts are:

proxy_http_version 1.1;


proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";

Without those, WebSocket connections commonly fail.

8. Verify the AI WebSocket exists

On the server, search your backend:

grep -R "ws/ai" -n backend_dashboard-main/app

Also:

grep -R "@.*websocket" -n backend_dashboard-main/app

You should find the actual FastAPI WebSocket route.

For example:

@router.websocket("/ws/ai")
async def ai_websocket(websocket: WebSocket):
    ...

Then confirm how that router is included in main.py.

9. Your current architecture should be

After fixing this, the production flow should be:

                    Internet
                       │
                       ▼
              https://droneyaan.com
                       │
                       ▼
                    Nginx
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
   React SPA       FastAPI REST    WebSockets
                    :8000
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
   CameraManager   AI Worker      Publisher
        │              │              │
        ▼              ▼              ▼
    MediaMTX       YOLO11m       Streaming
        │
        ▼
   RTSP Cameras

Your current error is happening around this section:

droneyaan.com
      │
      ▼
   Nginx
      │
      X  ← 502
      │
127.0.0.1:8000
10. Don't change the RTSP URL

This:

rtsp://13.201.184.38:8554/str

is not the cause of these 502 errors.

The errors are occurring before the camera API/WebSocket successfully reaches FastAPI.

Do these 4 commands now

On your EC2, run these one by one:

sudo ss -lntp | grep 8000
curl -i http://127.0.0.1:8000/cameras/
sudo nginx -t
sudo tail -50 /var/log/nginx/error.log

Paste the complete output of those four commands here. Also paste your current /etc/nginx/sites-enabled/... configuration and the relevant app/main.py include_router(...) lines. Then I can give you the exact Nginx configuration for droneyaan.com, including /cameras/, /ws/ai, and your publisher WebSocket, without breaking your existing deployment.
