name: Backend Deploy to GCP

on:
  push:
    branches: [ main ]
    paths:
      - 'backend/**'
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          cd backend
          pip install -r requirements.txt
          pip install pytest pytest-cov
      
      - name: Run tests
        run: |
          cd backend
          pytest --cov=. --cov-report=xml

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup GCP CLI
        uses: google-github-actions/setup-gcloud@v1
        with:
          service_account_key: ${{ secrets.GCP_SA_KEY }}
          project_id: ${{ secrets.GCP_PROJECT_ID }}
      
      - name: Deploy to Compute Engine
        run: |
          # VM에 코드 복사
          gcloud compute scp --recurse backend/ ${{ secrets.GCP_VM_NAME }}:~/app --zone=${{ secrets.GCP_ZONE }}
          
          # 배포 스크립트 실행
          gcloud compute ssh ${{ secrets.GCP_VM_NAME }} --zone=${{ secrets.GCP_ZONE }} --command="cd ~/app && chmod +x deploy.sh && ./deploy.sh"# don
