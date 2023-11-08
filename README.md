# Github Actions React S3

    https://www.youtube.com/watch?v=HVw_NZUhDKs

    https://www.youtube.com/watch?v=l97zYgiB57k

0. Crear usuario de aws con permiso total en S3 y guardar AWS_ACCESS_KEY_ID | AWS_SECRET_ACCESS_KEY

1. Crear bucket público con ACL allow

    react-app-gha

    Object Ownership - allow ACLs

    Bucket Policy

    {
        "Version": "2012-10-17",
        "Id": "Policy1568691946888",
        "Statement": [
            {
                "Sid": "Stmt1568691936002",
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:*",
                "Resource": "arn:aws:s3:::react-app-gha/*"
            }
        ]
    }
    
2. Secretos

    AWS_S3_BUCKET=react-app-gha
    AWS_ACCESS_KEY_ID=AKIAYTFWDFFBPXXZUCK7
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


4. Bucket

index.html file url

    https://react-app-gha.s3.amazonaws.com/index.html

Static website hosting

    http://react-app-gha.s3-website-us-east-1.amazonaws.com/