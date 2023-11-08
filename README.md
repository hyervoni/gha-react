# Github Actions React S3

## Iniciar React App

    npx create-react-app .

## Github Actions

    https://www.youtube.com/watch?v=HVw_NZUhDKs

    https://www.youtube.com/watch?v=l97zYgiB57k

    https://github.com/alelopezperez/areweleetcodeyet/blob/main/.github/workflows/main.yml

    https://antonputra.com/amazon/deploy-react-to-s3-and-cloudfront/#setup-cicd-pipeline-for-react-app-using-github-actions

1. Crear usuario de aws con permiso total en S3 y guardar AWS_ACCESS_KEY_ID | AWS_SECRET_ACCESS_KEY

2. Crear ROL OIDC en aws

        https://www.youtube.com/watch?v=k2Tv-EJl7V4

    AWS por seguridad ya no utiliza aws-access-key-id ni aws-secret-access-key

    * IAM add Idenitity Provider

        Provider: token.actions.githubusercontent.com
        Audience: sts.amazon.com
        Organization: none (luego se edita y se debe quitar en el json rol)

    * Creación del Rol en aws

        Crete Role, select web identity, select token.actions.githubusercontent.com and  sts.amazonaws.com
        Permission: AdministratorAccess, AmazonS3FullAccess
        Role Name: OIDC_GITHUB_CICD

        ARN: arn:aws:iam::590939892034:role/OIDC_GITHUB_CICD
        
        Edit Trusted entities to: 
 
            {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Principal": {
                            "Federated": "arn:aws:iam::590939892034:oidc-provider/token.actions.githubusercontent.com"
                        },
                        "Action": "sts:AssumeRoleWithWebIdentity",
                        "Condition": {
                            "StringLike": {
                                "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
                                "token.actions.githubusercontent.com:sub": "repo:hyervoni/*"
                            }
                        }
                    }
                ]
            }

    * Ajustar el workflow

            name: AWS Auth 
            uses: aws-actions/configure-aws-credentials@v4
            with:
            aws-region: us-east-1
            role-to-assume: arn:aws:iam::590939892034:role/OIDC_GITHUB_CICD
            role-session-name: S3Deployment
        

3. Crear bucket público

    Bucket: react-app-gha

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
    
4. Secretos

        AWS_S3_BUCKET=react-app-gha
        AWS_ACCESS_KEY_ID=AKIAYTFWDFFBPXXZUCK7
        AWS_SECRET_ACCESS_KEY=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

5. Workflow

        name: React CI

        on:
        push:
            branches:
            - main

        permissions:
        id-token: write
        contents: read

        jobs:
        build:
            runs-on: ubuntu-latest 

            steps: 

            - name: Checkout code
                uses: actions/checkout@v4

            - name: Setup Node.js
                uses: actions/setup-node@v3
                with:
                node-version: 18

            - name: Install dependencies
                run: npm ci --production

            - name: Build and test
                run: |
                npm run build
                npm test

            - name: AWS Auth 
                uses: aws-actions/configure-aws-credentials@v4
                with:
                aws-region: us-east-1
                role-to-assume: arn:aws:iam::590939892034:role/OIDC_GITHUB_CICD
                role-session-name: S3Deployment
                
            - name: Deploy to AWS S3
                env:
                AWS3: ${{ secrets.AWS_S3_BUCKET }}
                DIST: build
                run: |
                aws s3 sync --delete ${{ env.DIST }} s3://$AWS3

6. Bucket

index.html file url

    https://react-app-gha.s3.amazonaws.com/index.html

Static website hosting

    http://react-app-gha.s3-website-us-east-1.amazonaws.com/
