# Task 1 — Git Workflow

### Create Feature Branch
```bash
git checkout -b feature-update
```
Creates a separate branch for implementing and testing changes without directly modifying `main`.

### View Branches and Commit History
```bash
git branch
git log --oneline --graph --all
```
Displays available branches and the commit history for verifying the development workflow.

### Check Working Tree
```bash
git status
```
Checks whether all changes are committed and the working tree is clean.

# Task 2 — Maven Multi-Module Project

### Parent POM
**File:** `pom.xml`
```xml
<packaging>pom</packaging>
<modules>
    <module>course-registration-app</module>
</modules>
```
Defines the root project as a Maven parent and declares the application module.

### Child POM
**File:** `course-registration-app/pom.xml`
```xml
<parent>
    <groupId>com.university</groupId>
    <artifactId>course-registration-parent</artifactId>
    <version>1.0-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath>
</parent>
<artifactId>course-registration-app</artifactId>
<packaging>jar</packaging>
```
Configures the application module to inherit shared settings from the parent POM.

### Run Tests
```bash
mvn clean test
```
Cleans the project, compiles the source code, and runs the JUnit tests across the Maven modules.

### Package Application
```bash
mvn package
```
Builds the modules and packages the application as a JAR.

# Task 3 — Fix Course-Capacity Boundary Defect

### Validate Course Capacity
**File:** `RegistrationService.java`
```java
if (course.getCurrentEnrollment() >= course.getCapacity()) {
    return false;
}
```
Rejects registration when the course is full while allowing the final available seat to be filled.

### Test Capacity Boundary
**File:** `RegistrationServiceTest.java`
```java
assertFalse(service.register(student, fullCourse));
assertEquals(previousEnrollment, fullCourse.getCurrentEnrollment());

assertTrue(service.register(studentWithSpace, courseWithOneSeatRemaining));
assertEquals(courseWithOneSeatRemaining.getCapacity(),
             courseWithOneSeatRemaining.getCurrentEnrollment());
```
Verifies that full courses reject registration and the last available seat can be used.

# Task 4 — Case-Insensitive Duplicate Registration

### Case-Insensitive Course Lookup
**File:** `Student.java`
```java
public boolean isRegistered(String courseCode) {
    return registeredCourses.stream()
            .anyMatch(code -> code.equalsIgnoreCase(courseCode));
}
```
Detects duplicate course codes regardless of letter case.

### Test Case Variations
**File:** `RegistrationServiceTest.java`
```java
assertFalse(service.register(student, lowerCaseCourse));
assertFalse(service.register(student, mixedCaseCourse));
assertEquals(previousCredits, student.getTotalCredits());
assertEquals(previousEnrollment, course.getCurrentEnrollment());
```
Verifies that case variations of an existing course are rejected without changing credits or enrollment.

# Task 5 — Jenkins Declarative Pipeline

### Jenkinsfile
**File:** `Jenkinsfile`
```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Test') {
            steps {
                bat 'mvn clean test'
            }
            post {
                always {
                    junit 'course-registration-app/target/surefire-reports/TEST-*.xml'
                }
            }
        }
        stage('Package') {
            steps {
                bat 'mvn package'
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'course-registration-app/target/*.jar',
                             fingerprint: true
        }
    }
}
```
Defines checkout, test, report publication, packaging, and successful-build artifact archiving for a Windows Jenkins agent.

# Task 6 — Kubernetes Deployment Using Minikube

### Deployment Manifest
**File:** `k8s/deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
```
Deploys two Nginx replicas and labels the Pods so the Deployment can manage them.

### NodePort Service
**File:** `k8s/service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```
Exposes the Nginx Pods through a NodePort Service on port `30080`.

### Apply Kubernetes Manifests
```powershell
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl get deployments
kubectl get pods
kubectl get services
```
Creates the Deployment and Service and displays their current status.

### Access Nginx
```powershell
minikube service nginx-service --url
```
Returns a URL for opening the Nginx application in a browser.

### Scale Deployment
```powershell
kubectl scale deployment nginx-deployment --replicas=3
kubectl rollout status deployment/nginx-deployment
kubectl get pods
```
Scales the Deployment to three replicas and verifies that the rollout completes.

### Verify Self-Healing
```powershell
kubectl delete pod <POD_NAME>
kubectl get pods -w
```
Deletes one Pod and watches Kubernetes create a replacement to maintain the desired replica count.

# Task 7 — Documentation and Final Verification

### Verify Git Status
```bash
git status
git log --oneline -5
```
Checks the final working-tree state and displays the latest commits for submission verification.
