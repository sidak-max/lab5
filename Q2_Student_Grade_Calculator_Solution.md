# Question 2 – Student Grade Calculator (Kubernetes CI/CD)

- Push 1: total and average marks → first deployment
- Push 2: grade calculation → updated deployment

Replace `YOUR_USERNAME` (GitHub) and `YOUR_DOCKERHUB_USERNAME` (Docker Hub) everywhere.
All commands are for the VS Code PowerShell terminal.

Do not run `git push` anywhere except Push 1 and Push 2 below.

---

## Part A – Preparation (no push)

### A1. Verify tools

```powershell
git --version
python --version
docker --version
docker info
kubectl version --client
kubectl get nodes
```

`kubectl get nodes` must show `Ready`.

### A2. Create GitHub repository (empty)

GitHub → **New repository** → name `grade-calculator` → **Public** → do not add README/.gitignore/license → **Create repository**.

### A3. Create Docker Hub repository and token

1. Docker Hub → **Create repository** → name `grade-calculator` → **Public**.
2. Docker Hub → **Account Settings → Personal access tokens → Generate new token** → **Read & Write** → copy it.

### A4. Add GitHub secrets

GitHub repo → **Settings → Secrets and variables → Actions → New repository secret**:

| Name | Value |
|---|---|
| `DOCKERHUB_USERNAME` | your Docker Hub username |
| `DOCKERHUB_TOKEN` | the access token |

### A5. Create project

```powershell
mkdir grade-calculator
cd grade-calculator
mkdir k8s
mkdir .github\workflows
New-Item app.py, requirements.txt, test_app.py, Dockerfile, .dockerignore, .gitignore
New-Item k8s\deployment.yaml, k8s\service.yaml, .github\workflows\pipeline.yml
code .
```

---

## Part B – Push 1 files (total and average)

### app.py

```python
from flask import Flask, request, render_template_string

app = Flask(__name__)

PAGE = """
<!doctype html>
<html>
<head><title>Student Grade Calculator</title></head>
<body>
<h2>Student Grade Calculator</h2>
<form method="post">
<input name="marks" placeholder="e.g. 80 90 70" required>
<button type="submit">Calculate</button>
</form>
{% if error %}<p>{{ error }}</p>{% endif %}
{% if result %}
<p>Total: {{ result.total }}</p>
<p>Average: {{ result.average }}</p>
{% endif %}
</body>
</html>
"""


def parse_marks(text):
    return [float(m) for m in text.replace(",", " ").split()]


def calculate_total(marks):
    return round(sum(marks), 2)


def calculate_average(marks):
    return round(sum(marks) / len(marks), 2)


@app.route("/", methods=["GET", "POST"])
def index():
    result = None
    error = None
    if request.method == "POST":
        try:
            marks = parse_marks(request.form["marks"])
            if not marks or any(m < 0 or m > 100 for m in marks):
                raise ValueError
            result = {
                "total": calculate_total(marks),
                "average": calculate_average(marks),
            }
        except ValueError:
            error = "Enter valid marks between 0 and 100, separated by spaces or commas."
    return render_template_string(PAGE, result=result, error=error)


@app.route("/health")
def health():
    return {"status": "healthy"}, 200


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

### requirements.txt

```
Flask>=3.0,<4.0
pytest>=8.0,<9.0
```

### test_app.py

```python
from app import app, calculate_total, calculate_average


def test_total():
    assert calculate_total([80, 90, 70]) == 240


def test_average():
    assert calculate_average([80, 90, 70]) == 80


def test_home_page():
    client = app.test_client()
    response = client.get("/")
    assert response.status_code == 200
    assert b"Student Grade Calculator" in response.data


def test_calculate_post():
    client = app.test_client()
    response = client.post("/", data={"marks": "80 90 70"})
    assert response.status_code == 200
    assert b"Total: 240.0" in response.data
    assert b"Average: 80.0" in response.data


def test_invalid_marks():
    client = app.test_client()
    response = client.post("/", data={"marks": "abc"})
    assert b"Enter valid marks" in response.data


def test_health_endpoint():
    client = app.test_client()
    response = client.get("/health")
    assert response.status_code == 200
    assert response.json["status"] == "healthy"
```

### Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

### .dockerignore

```
.venv/
__pycache__/
.pytest_cache/
.git/
```

### .gitignore

```
.venv/
__pycache__/
.pytest_cache/
.env
```

### .github/workflows/pipeline.yml

```yaml
name: Grade Calculator CI/CD

on:
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  test:
    name: Checkout and Test
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run automated tests
        run: python -m pytest -v

  docker:
    name: Build and Push Docker Image
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/grade-calculator:${{ github.sha }}
```

### k8s/deployment.yaml

Leave `IMAGE_PLACEHOLDER` exactly as written. It is replaced at deploy time so the manifest never needs an extra push.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grade-calculator
spec:
  replicas: 2
  selector:
    matchLabels:
      app: grade-calculator
  template:
    metadata:
      labels:
        app: grade-calculator
    spec:
      containers:
        - name: grade-calculator
          image: IMAGE_PLACEHOLDER
          ports:
            - containerPort: 5000
          readinessProbe:
            httpGet:
              path: /health
              port: 5000
            initialDelaySeconds: 5
            periodSeconds: 5
```

### k8s/service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: grade-calculator
spec:
  type: NodePort
  selector:
    app: grade-calculator
  ports:
    - port: 80
      targetPort: 5000
      nodePort: 30080
```

---

## Part C – Test locally before Push 1 (no push)

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m pytest -v
.\.venv\Scripts\python.exe app.py
```

Open http://127.0.0.1:5000/ → enter `80 90 70` → shows `Total: 240.0` and `Average: 80.0`. Stop with `Ctrl+C`.

Optional Docker check:

```powershell
docker build -t grade-calculator:test .
docker run --rm -d --name grade-test -p 5000:5000 grade-calculator:test
docker stop grade-test
```

---

## Part D – Push 1 (Version Control → CI → Build)

```powershell
git init
git add .
git commit -m "Grade calculator: total and average"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/grade-calculator.git
git push -u origin main
```

GitHub → **Actions** → wait until both jobs `Checkout and Test` and `Build and Push Docker Image` are green.
Docker Hub → `grade-calculator` → **Tags** → a tag equal to the commit SHA is present.

📸 Screenshots: green Actions run, Docker Hub tag.

---

## Part E – Deploy Push 1 image to Kubernetes

```powershell
kubectl config current-context
kubectl get nodes
```

Context must be `docker-desktop`.

```powershell
$DH = "YOUR_DOCKERHUB_USERNAME"
$SHA = git rev-parse HEAD
$SHA
```

Confirm `$SHA` matches the tag in Docker Hub. Deploy:

```powershell
(Get-Content k8s\deployment.yaml -Raw).Replace("IMAGE_PLACEHOLDER", "$DH/grade-calculator:$SHA") | kubectl apply -f -
kubectl apply -f k8s\service.yaml
kubectl rollout status deployment/grade-calculator
kubectl get deployments
kubectl get pods
kubectl get services
kubectl get deployment grade-calculator -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Expected: 2/2 ready, 2 pods `Running`, service `80:30080/TCP`, and the last command prints the image tagged with the Push 1 SHA.

Open http://localhost:30080/ → enter `80 90 70` → only **Total** and **Average** are shown (no grade).

📸 Screenshots: kubectl outputs, image tag, browser result.

---

## Part F – Modify the application for Push 2 (grade)

Replace the whole of `app.py`:

```python
from flask import Flask, request, render_template_string

app = Flask(__name__)

PAGE = """
<!doctype html>
<html>
<head><title>Student Grade Calculator</title></head>
<body>
<h2>Student Grade Calculator</h2>
<form method="post">
<input name="marks" placeholder="e.g. 80 90 70" required>
<button type="submit">Calculate</button>
</form>
{% if error %}<p>{{ error }}</p>{% endif %}
{% if result %}
<p>Total: {{ result.total }}</p>
<p>Average: {{ result.average }}</p>
<p>Grade: {{ result.grade }}</p>
{% endif %}
</body>
</html>
"""


def parse_marks(text):
    return [float(m) for m in text.replace(",", " ").split()]


def calculate_total(marks):
    return round(sum(marks), 2)


def calculate_average(marks):
    return round(sum(marks) / len(marks), 2)


def calculate_grade(average):
    if average >= 90:
        return "A"
    if average >= 80:
        return "B"
    if average >= 70:
        return "C"
    if average >= 60:
        return "D"
    return "F"


@app.route("/", methods=["GET", "POST"])
def index():
    result = None
    error = None
    if request.method == "POST":
        try:
            marks = parse_marks(request.form["marks"])
            if not marks or any(m < 0 or m > 100 for m in marks):
                raise ValueError
            average = calculate_average(marks)
            result = {
                "total": calculate_total(marks),
                "average": average,
                "grade": calculate_grade(average),
            }
        except ValueError:
            error = "Enter valid marks between 0 and 100, separated by spaces or commas."
    return render_template_string(PAGE, result=result, error=error)


@app.route("/health")
def health():
    return {"status": "healthy"}, 200


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

Replace the whole of `test_app.py`:

```python
from app import app, calculate_total, calculate_average, calculate_grade


def test_total():
    assert calculate_total([80, 90, 70]) == 240


def test_average():
    assert calculate_average([80, 90, 70]) == 80


def test_grade_boundaries():
    assert calculate_grade(95) == "A"
    assert calculate_grade(90) == "A"
    assert calculate_grade(85) == "B"
    assert calculate_grade(75) == "C"
    assert calculate_grade(65) == "D"
    assert calculate_grade(40) == "F"


def test_home_page():
    client = app.test_client()
    response = client.get("/")
    assert response.status_code == 200
    assert b"Student Grade Calculator" in response.data


def test_calculate_post():
    client = app.test_client()
    response = client.post("/", data={"marks": "80 90 70"})
    assert response.status_code == 200
    assert b"Total: 240.0" in response.data
    assert b"Average: 80.0" in response.data
    assert b"Grade: B" in response.data


def test_invalid_marks():
    client = app.test_client()
    response = client.post("/", data={"marks": "abc"})
    assert b"Enter valid marks" in response.data


def test_health_endpoint():
    client = app.test_client()
    response = client.get("/health")
    assert response.status_code == 200
    assert response.json["status"] == "healthy"
```

Test locally (no push):

```powershell
.\.venv\Scripts\python.exe -m pytest -v
```

All tests must pass.

---

## Part G – Push 2 (updated version → new pipeline run)

```powershell
git add .
git commit -m "Grade calculator: add grade calculation"
git push
```

GitHub → **Actions** → new run → both jobs green.
Docker Hub → a second, different SHA tag is present.

```powershell
$SHA2 = git rev-parse HEAD
$SHA2
```

---

## Part H – Deploy Push 2 image

```powershell
kubectl set image deployment/grade-calculator grade-calculator=$DH/grade-calculator:$SHA2
kubectl rollout status deployment/grade-calculator
kubectl get pods
kubectl get deployment grade-calculator -o jsonpath="{.spec.template.spec.containers[0].image}"
```

If PowerShell was closed after Part E, set `$DH` again first:

```powershell
$DH = "YOUR_DOCKERHUB_USERNAME"
$SHA2 = git rev-parse HEAD
```

Open http://localhost:30080/ → hard refresh (`Ctrl+F5`) → enter `80 90 70` → shows:

```
Total: 240.0
Average: 80.0
Grade: B
```

Try `95 92 98` → `Grade: A`.

📸 Screenshots: green Actions run (Push 2), Docker Hub tags, rollout output, new image tag, browser with grade.

---

## Troubleshooting

```powershell
kubectl get all
kubectl describe pod
kubectl logs deployment/grade-calculator
kubectl get events --sort-by=.metadata.creationTimestamp
```

| Problem | Fix |
|---|---|
| Actions docker job fails at login | Check secret names `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`, token has Read & Write |
| `ImagePullBackOff` | Image name or SHA wrong, Docker Hub repo not Public, or the Actions run not finished |
| Pod `Pending` / `CrashLoopBackOff` | `kubectl describe pod`, `kubectl logs deployment/grade-calculator` |
| Browser cannot connect | `kubectl get pods` (ready?), `kubectl get services` (30080) |
| Old page after Push 2 | Confirm new SHA in image command, rollout finished, `Ctrl+F5` |
| `git push` rejected | Remote repo must be empty; check `git remote -v` |

---

## Final checklist

- [ ] Python app in VS Code (Source Code)
- [ ] Exactly two pushes (Version Control)
- [ ] Actions checks out code, installs dependencies, runs tests (CI)
- [ ] Docker image pushed to Docker Hub with a unique SHA tag per push (Build)
- [ ] Kubernetes Deployment + NodePort Service (Deployment)
- [ ] Push 1 deployed: total and average only
- [ ] Push 2 deployed: grade added, image tag differs from Push 1
