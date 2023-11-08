# Github Actions React S3

    https://www.youtube.com/watch?v=HVw_NZUhDKs

1. Crear bucket público 

    react-app-gha

2. Secretos

    AWS_S3_BUCKET=react-app-gha
    AWS_ACCESS_KEY_ID=AKIAYTFWDFFBHHWTYBME
    AWS_SECRET_ACCESS_KEY=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

3. Workflow

    name: React CI

    on:
    push:
        branches:
        - "main"

    jobs:
    build:
        runs-on: ubuntu-latest
        strategy:
        matrix:
            node-version: [15.x]

        steps:

        - uses: actions/checkout@v2
        - run: npm install
        - run: npm run build
        - run: npm test
        
        - uses: jakejarvis/s3-sync-action@master
            with:
            args: --acl public-read --follow-symlinks --delete
            env:
            AWS_S3_BUCKET: ${{ secrets.AWS_S3_BUCKET }}
            AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
            AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
            AWS_REGION: 'us-east-1'   
            SOURCE_DIR: 'build'   