# Lab 11: Containerization (own Docker image + Docker Hub + nginx on Kubernetes)

Source: `week11_devops_lab/lab-11-containerization.docx` (+ starter files in `lab11-Docker/`).
Two parts, plus the Docker Hub push you mentioned:

| Part | What you do | Tools |
|---|---|---|
| I | Write a Dockerfile, build an image called `hellodocker`, run it, hit it on `localhost:4000` | Docker Desktop |
| I+ | Tag that image with your Docker Hub username and push it | Docker Hub |
| II | Fork the instructor's nginx repo, `kubectl apply` a Deployment + NodePort Service, reach nginx through Minikube, clean up | git, kubectl, Minikube |

> 📌 The doc itself does **not** mention Docker Hub. If your prof said it in class, it's
> probably "push the image you built in Part I". That's what section I+ does. If the
> requirement turns out to be something else (e.g. deploy *your* image to k8s), tell me and
> I'll add it.

## Deliverable

Same pattern as your earlier labs: `Week11_Lab_Exercise_101572409.docx`, one heading per
screenshot (rename if the D2L dropbox says otherwise). The doc says "take screenshots for
each step", so:

| # | Part | Screenshot of |
|---|---|---|
| 1 | I | `docker run hello-world` (setup check) |
| 2 | I | Your `lab11_Docker` folder listing + the Dockerfile open in VS Code |
| 3 | I | `docker build --tag=hellodocker .` finishing successfully |
| 4 | I | `docker image ls` showing `hellodocker` |
| 5 | I | Browser at `http://localhost:4000` ("Hello Ben!" + Redis error) |
| 6 | I | `curl.exe http://localhost:4000` in a second terminal |
| 7 | I | `docker container ls` then `docker container stop <id>` |
| 8 | I+ | `docker login` succeeded + `docker push` output |
| 9 | I+ | Your repo page on hub.docker.com showing the `1.0` tag |
| 10 | II | Your fork on GitHub + `git clone` in the terminal |
| 11 | II | `kubectl apply -f ...` (both files) + `kubectl rollout status` |
| 12 | II | `kubectl get pods -l app=nginx -o wide` + `kubectl get svc nginx-nodeport` |
| 13 | II | `minikube service nginx-nodeport --url` + nginx welcome page (browser or curl) |
| 14 | II | Cleanup: both `kubectl delete -f ...` commands |

## What's already set up (from lab 10 on 2026-11-15)

| Thing | Status |
|---|---|
| Docker Desktop 29.8.2 (WSL 2) | Installed, **start it manually** before anything else |
| Minikube v1.39.0 | Cluster created in lab 10; `minikube start` resumes it |
| kubectl | Comes with Docker Desktop |
| Python on your PC | **Not needed.** That's the whole point of Part I 😎 |

## Files I made for you

```
3104-devops\lab11\
├── lab11-guide.md          ← this file
└── lab11_Docker\           ← the build context (the "." in docker build .)
    ├── Dockerfile          ← the lab's Dockerfile, commented, NAME set to Ben
    ├── .dockerignore       ← keeps screenshots/docx/md out of the image
    ├── app.py              ← copied from the starter (correct indentation)
    └── requirements.txt    ← Flask, Redis
```

I ran `app.py` here with Flask and it returns
`Hello Ben! … Visits: cannot connect to Redis, counter disabled`, which is the expected
output. I couldn't run the full `docker build` from my side (my sandbox can't reach PyPI
from inside a build), so your build is the first real one.

The doc wants you to create the folder in your home directory with `touch`. Doing it in
your repo instead is fine: what's graded is the screenshots, not the path. If you want to
type the files yourself for practice, delete mine and follow the doc; contents are identical
apart from the comments, `NAME`, and the two fixes below.

### Two bugs in the doc you should know about

1. **`app.py` in the doc is mis-indented.** The `return html.format(...)` line is flush-left
   in the docx, which makes it outside the function → `SyntaxError: 'return' outside function`.
   It must be indented 4 spaces (inside `def hello():`). The starter file and my copy are correct.
2. **The doc says the image uses Python 2.7.** It doesn't. `FROM python:3.11-slim` = Python 3.11.
   Leftover text from an old version of the lab.

---

## Part I: Build and run your own image

All commands are **PowerShell**, run from the folder in each step.

### Step 1: Start Docker and sanity check (screenshot 1)

Start menu → **Docker Desktop**, wait for *Engine running*. Then:

```powershell
docker run hello-world   # pulls a tiny image, prints "Hello from Docker!", exits
```

### Step 2: Look at what you're building (screenshot 2)

```powershell
cd ~\Desktop\class-projects\y3-fall\3104-devops\lab11\lab11_Docker
dir                      # Dockerfile, app.py, requirements.txt (.dockerignore is hidden-ish)
code .                   # open in VS Code, screenshot the Dockerfile
```

How the three files connect (this is the "why" the doc spends two pages on):

```
Dockerfile ── FROM python:3.11-slim ──► gives you Python, no install on your PC
     │
     ├── COPY . /app ─────────────────► puts app.py + requirements.txt in the image
     ├── RUN pip install -r requirements.txt ─► installs Flask + redis INSIDE the image
     ├── ENV NAME="Ben" ──────────────► app.py reads os.getenv("NAME")
     └── CMD python app.py ───────────► Flask starts on 0.0.0.0:80 inside the container
```

### Step 3: Build the image (screenshot 3)

```powershell
docker build --tag=hellodocker .
#             └─ name of the image   └─ "." = build context = this folder
```

First build takes a minute (it downloads `python:3.11-slim`). It ends with something like
`naming to docker.io/library/hellodocker:latest`. The doc's screenshot says
"Successfully built"; that's the old builder's wording. BuildKit's output looks different, same result.

### Step 4: See it in your local image list (screenshot 4)

```powershell
docker image ls          # hellodocker   latest   <id>   ...   ~150MB
```

### Step 5: Run it (screenshot 5)

```powershell
docker run -p 4000:80 hellodocker
#           └ host port 4000 ──► container port 80 (where Flask listens)
```

This **stays in the foreground** and prints Flask's log. Leave it running. Open
`http://localhost:4000` in your browser. You should see:

```
Hello Ben!
Hostname: 3f2a9c1b7d4e        ← the container ID, from socket.gethostname()
Visits: cannot connect to Redis, counter disabled
```

The Redis error is **expected**. The code tries to reach a Redis server at hostname
`redis`, and there isn't one (we only installed the Python client library). 
⏳ **The page can take 30-45 seconds to load. That's normal here.** Looking up the hostname
`redis` is slow inside Docker Desktop, the 2-second `socket_connect_timeout` in `app.py`
doesn't cover that lookup, and the Redis client retries. Wait it out; don't restart the
container. (Measured on your PC: 44 s.) Next lab adds a real Redis and it becomes instant.

### Step 6: Curl it from a second terminal (screenshot 6)

Open a **new** PowerShell window (the first one is busy running the container):

```powershell
curl.exe http://localhost:4000
```

⚠️ Type `curl.exe`, not `curl`. In Windows PowerShell, `curl` is an alias for
`Invoke-WebRequest`, which prints a wall of object properties instead of the HTML.

### Step 7: Stop the container (screenshot 7)

Go back to the first terminal and press `Ctrl+C`. Then confirm and clean up:

```powershell
docker container ls                     # running containers only
# If you still see hellodocker listed:
docker container stop <CONTAINER_ID>    # first 3-4 chars of the ID are enough
```

On Windows, `Ctrl+C` sometimes detaches without stopping the container. That's why the
doc has you check with `docker container ls`.

---

## Part I+: Push your image to Docker Hub

**Concept:** your image currently lives only in Docker Desktop's local store on your PC.
Docker Hub is a public registry (GitHub, but for images). To push, the image name must
start with your Docker Hub username: `<username>/<repo>:<tag>`. `hellodocker` alone means
`docker.io/library/hellodocker`, and `library/` is reserved for official images, so pushing
it as-is gets you `denied: requested access to the resource is denied`.

Your Docker Hub username is **`7harlot`** (already filled in below; usernames are lowercase, exactly as on
hub.docker.com, not your email).

### Step 1: Create the account + an access token

1. Sign up at https://hub.docker.com (free Personal plan).
2. Avatar → **Account settings → Personal access tokens → Generate new token**.
   Name it `comp3104-lab11`, permissions **Read & Write**. Copy it now, it's shown once.

Why a token instead of your password: if it leaks (screenshot, shell history), you revoke
just that token. Your password stays safe. Docker also requires a token if you turn on 2FA.

### Step 2: Log in (screenshot 8, part 1)

```powershell
docker login -u 7harlot
# Password: <paste the access token, nothing shows as you paste, that's normal>
# → Login Succeeded
```

If Docker Desktop is already signed in to Hub (top-right of the Desktop app), this
succeeds without asking. 🔒 Don't screenshot the token. The `Login Succeeded` line is enough.

### Step 3: Tag the image with your username

```powershell
docker tag hellodocker 7harlot/hellodocker:1.0
docker tag hellodocker 7harlot/hellodocker:latest
docker image ls          # same IMAGE ID appears 3 times, a tag is just another name
```

Why `1.0` *and* `latest`: `latest` is what people get if they don't specify a tag.
`1.0` is a pinned version you can always roll back to. Real projects always push a
version tag, never only `latest`.

### Step 4: Push (screenshot 8, part 2)

```powershell
docker push 7harlot/hellodocker:1.0
docker push 7harlot/hellodocker:latest   # fast, layers already uploaded
```

You'll see each layer upload, then a `digest: sha256:...` line. The repo is created
automatically as **public** on first push.

### Step 5: Prove it works from the registry (screenshot 9)

Open `https://hub.docker.com/r/7harlot/hellodocker` and screenshot the **Tags** tab.

Bonus proof that the image really lives on Hub (nice to have, not required):

```powershell
docker rmi 7harlot/hellodocker:1.0               # delete the local copy of that tag
docker run -p 4000:80 7harlot/hellodocker:1.0    # Docker pulls it back from Hub
```

| Error | Cause | Fix |
|---|---|---|
| `denied: requested access to the resource is denied` | Image not prefixed with your username, or username typo/case | Re-run Step 3 with exact lowercase username |
| `unauthorized: incorrect username or password` | Used email as username, or token is read-only | Use the username; make a Read & Write token |
| `An image does not exist locally with the tag` | Typo in tag | `docker image ls` and copy the name |

---

## Part II: nginx on Kubernetes (Minikube)

**Concept:** in Part I *you* ran one container by hand. Here you describe what you want in
YAML and Kubernetes keeps it that way:

- **Deployment** (`nginx-deploy.yaml`): "keep N nginx pods running". Kill one and k8s makes another.
- **Service** of type **NodePort** (`nginx-service.yaml`): a stable address in front of
  those pods. It finds them by the label `app=nginx` and opens port **30080** on the node.

### Step 1: Fork + clone (screenshot 10)

1. Fork https://github.com/pritamworld/DevOps_Kubernetes_exercise9 to your account (7harlot).
2. Clone it **beside** your 3104-devops repo, not inside it (a repo inside a repo turns into
   a confusing "embedded repository" in git status):

```powershell
cd ~\Desktop\class-projects\y3-fall
git clone https://github.com/7harlot/DevOps_Kubernetes_exercise9.git
cd DevOps_Kubernetes_exercise9
dir                      # expect nginx-deploy.yaml and nginx-service.yaml
```

> I couldn't open that repo from my side (network block), so the commands below follow the
> lab doc. If the file names in your clone differ, use the real names, and tell me if the
> service isn't called `nginx-nodeport`.

### Step 2: Start the cluster

```powershell
minikube start           # resumes the lab-10 cluster (Docker Desktop must be running)
kubectl get nodes        # minikube   Ready
```

### Step 3: Apply and verify (screenshots 11 + 12)

```powershell
kubectl apply -f nginx-deploy.yaml      # → deployment.apps/nginx created
kubectl apply -f nginx-service.yaml     # → service/nginx-nodeport created
kubectl rollout status deployment/nginx # waits until every pod is Ready
kubectl get pods -l app=nginx -o wide   # -l = filter by label, -o wide = show pod IPs + node
kubectl get svc nginx-nodeport          # TYPE NodePort, PORT(S) 80:30080/TCP
```

### Step 4: Reach nginx (screenshot 13)

```powershell
minikube service nginx-nodeport --url
# → http://127.0.0.1:4666   (example: the port is random, copy YOURS)
```

⚠️ **Important Windows gotcha.** With Minikube's Docker driver on Windows, the Minikube IP
(`192.168.49.2`) is inside Docker's network and **your PC can't reach it directly**. So the
doc's second option (`curl http://$MINIKUBE_IP:30080`) will just hang on your machine.
`minikube service --url` opens a tunnel instead, and **that terminal must stay open** while
you use the URL. In a second terminal:

```powershell
curl.exe http://127.0.0.1:4666          # replace 4666 with the port YOUR tunnel printed → "Welcome to nginx!"
```

Or paste the URL in your browser and screenshot the nginx welcome page. `Ctrl+C` the tunnel
when done.

For reference, the doc's `MINIKUBE_IP=$(minikube ip)` is bash syntax. The PowerShell version
is `$MINIKUBE_IP = minikube ip; curl.exe "http://${MINIKUBE_IP}:30080"`. It only works from
inside the Minikube network (e.g. `minikube ssh`), for the reason above.

### Step 5: Clean up (screenshot 14)

```powershell
kubectl delete -f nginx-deploy.yaml
kubectl delete -f nginx-service.yaml
kubectl get all          # only service/kubernetes should remain
minikube stop            # frees ~2 GB RAM; cluster stays for next lab
```

---

## Git (you run these, when you're happy)

Your 3104-devops repo already has untracked `lab07/`, `lab10/`, and `week11_devops_lab/`.
Suggested:

```powershell
cd ~\Desktop\class-projects\y3-fall\3104-devops
git add lab11/
git commit -m "lab11: Dockerfile + Flask app, Docker Hub push guide"
git push
```

`week11_devops_lab/` is the instructor's material (1 MB docx). I'd leave it out of the repo
or add it to `.gitignore`. Your call.

## Quick recap: what each command taught you

| Command | Lesson |
|---|---|
| `docker build -t name .` | Dockerfile + context → image |
| `docker run -p host:container` | image → running container, with a port bridge |
| `docker tag` / `docker push` | images are just named layers; registries share them |
| `kubectl apply -f` | describe desired state, k8s makes it real |
| Service selector `app=nginx` | labels are how k8s wires services to pods |
