CI/CD Pipeline Using GitHub 
`.github/workflows/ci-cd.yml`

```yaml
name: CI-CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  IMAGE_NAME: yourhubuser/task-app
  JAVA_VERSION: '17'

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up JDK ${{ env.JAVA_VERSION }}
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: ${{ env.JAVA_VERSION }}

      - name: Cache Maven packages
        uses: actions/cache@v3
        with:
          path: '~/.m2/repository'
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
          restore-keys: |
            ${{ runner.os }}-m2-

      - name: Build & test with Maven
        run: mvn -B clean verify

      - name: Build Docker image
        run: |
          docker build -t $IMAGE_NAME:${{ github.sha }} .
          echo $IMAGE_NAME

      - name: Log in to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USER }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Push Docker image
        run: |
          docker push $IMAGE_NAME:${{ github.sha }}
          docker tag $IMAGE_NAME:${{ github.sha }} $IMAGE_NAME:latest
          docker push $IMAGE_NAME:latest
```
