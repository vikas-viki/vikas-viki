
# 🛠️ Problems I Solved

A collection of engineering problems I faced (and solved) while building production-ready systems at a startup scale. From WebRTC pitfalls to deployment quirks — these are lessons worth documenting.

---

## 📡 1. Real-Time Video & Audio in a Virtual World (Gather Town–like)

### 🧩 The Problem
While building a 2D virtual environment platform like **Gather Town**, I initially chose **WebRTC** for real-time audio/video. However, I ran into limitations due to WebRTC’s **mesh topology** — which results in **N² connections** (each peer connects to every other peer). This doesn't scale well when multiple participants are active in the same room.

### 🚀 The Solution: Mediasoup + Next.js

To overcome the mesh bottleneck, I switched to **[mediasoup](https://mediasoup.org/)**, which provides an SFU (Selective Forwarding Unit) architecture.

#### 🎯 How Mediasoup Works (Core Summary)

- Mediasoup uses **SFU topology**, where each client sends and receives media through a central server.
- Internally, it handles:
  - **WebRTC Transports**: Manage ICE negotiation (DTLS/SRTP setup)
  - **Producers**: Created when a client sends media to the server
  - **Consumers**: Created when a client receives media from the server
  - **Routers**: Media routers that connect transports and manage RTP flow
- This drastically reduces peer load (only 1 upstream + 1 downstream per client).

✅ Result: Smooth, scalable real-time audio/video in a multiplayer virtual space.

---

## 💾 2. Building & Deploying a Next.js App on a 2GB DigitalOcean Droplet

### 🧩 The Problem
For a small-scale startup, I had a **2GB RAM Droplet** hosting both the frontend and backend via **NGINX reverse proxy**. Due to limited memory, building Next.js on the server was not viable — it would crash or slow everything down.

### 🚀 The Solution: CI/CD Build + Targz + PM2 Zero Downtime

- I shifted the build process to GitHub Actions (CI/CD).
- After building, I `rsync`-ed the `.next`, `public`, and config files to the server.
- 🧨 **Issue**: Since `node_modules` on CI and Droplet differed (i used to download packes in server), **Next.js dynamic images wouldn't load properly**.
- ✅ Fix:
  - I zipped the build output **along with the `node_modules`** using `tar.gz`
  - Transferred the tarball to the droplet
  - Extracted and **swapped folders using atomic symlinks**
  - Restarted the app with `pm2 reload` for **zero downtime**

📦 Now the server gets the exact build + dependencies from CI — identical to local behavior, no surprises.

---

## 🧠 Lessons

- WebRTC is great — until it's not. Know your topology.
- Mediasoup offers just the right amount of control for scalable media.
- Building on server = 💣 if your memory is tight.
- Deployment needs to be **atomic**, **reproducible**, and **predictable**.

---

🎯 These learnings were part of real-world freelance and startup projects — and each bug led to a better design.
