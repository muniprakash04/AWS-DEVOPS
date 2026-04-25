name: Deploy to S3

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      # Optional: install dependencies & build (for React, Vue, etc.)
      - name: Install dependencies
        run: npm install

      - name: Build project
        run: npm run build

      # Configure AWS credentials
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-access-key: ${{ secrets.AWS_ACCESS_KEY }}
          aws-region: ap-south-1

      # Sync files to S3
      - name: Deploy to S3 bucket
        run: |
          aws s3 sync ./build s3://muniprakash --delete

      # Optional: Invalidate CloudFront cache
      - name: Invalidate CloudFront
        if: success()
        run: |
          aws cloudfront create-invalidation \
            --distribution-id YOUR_DISTRIBUTION_ID \
            --paths "/*"
