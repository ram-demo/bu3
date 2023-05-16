name: Deploy
on:
  push:
    branches:
      - qa
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
       node-version: [16.x]    
    steps:
      - uses: actions/checkout@master
        with:
          node-version: ${{ matrix.node-version }}
      - name: Decrypt large secret
        run: gpg --quiet --batch --yes --decrypt --passphrase="$LARGE_SECRET_PASSPHRASE" --output /home/runner/work/email-service/email-service/.env /home/runner/work/email-service/email-service/.github/qa.gpg
        env:
          LARGE_SECRET_PASSPHRASE: ${{ secrets.LARGE_SECRET_PASSPHRASE }}

      - name: Access for gitmodules
        env:
          SSH_KEY_GITMODULES: ${{secrets.SSH_KEY_GITMODULES}}
        run: |
          mkdir $HOME/.ssh
          echo "$SSH_KEY_GITMODULES" > $HOME/.ssh/id_rsa
          chmod 400 $HOME/.ssh/id_rsa   
          cat $HOME/.ssh/id_rsa

      - name: submodule init
        run: git submodule init
      - name: update --init
        run: git submodule update --init 
      - working-directory: ./src/database/db-models
        run: git checkout qa  
      - working-directory: ./src/common-services
        run: git checkout qa
      - run: npm install    
      - run: npm run build 
      - run: cp package.json build
      - run: cp package-lock.json build
      - run: cp .env build
      - run: cp Dockerfile build

      - name: upload image to ECR
        uses: mr-smithers-excellent/docker-build-push@v5
        with:
          image: email
          tags: ${{ github.sha }}
          registry: 834772772058.dkr.ecr.us-east-1.amazonaws.com
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.QA_AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.QA_AWS_SECRET_ACCESS_KEY }}


      - name: Deploy to Kubernetes cluster
        uses: kodermax/kubectl-aws-eks@master
        env:
          RELEASE_IMAGE: 834772772058.dkr.ecr.us-east-1.amazonaws.com/email:${{ github.sha }}
          KUBE_CONFIG_DATA: ${{ secrets.QA_KUBE_CONFIG_DATA }}
          AWS_ACCESS_KEY_ID: ${{ secrets.QA_AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.QA_AWS_SECRET_ACCESS_KEY }}
          KUBECTL_VERSION: "v1.22.0"
          IAM_VERSION: "0.5.4"
          KUBE_NAMESPACE: qa-email-ns
        with:
          args: set image deployment/qa-email-deployment qa-email-container=${{ env.RELEASE_IMAGE }} --record -n $KUBE_NAMESPACE

      - name: verify deployment
        uses: kodermax/kubectl-aws-eks@master
        env:
          KUBE_CONFIG_DATA: ${{ secrets.QA_KUBE_CONFIG_DATA }}
          AWS_ACCESS_KEY_ID: ${{ secrets.QA_AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.QA_AWS_SECRET_ACCESS_KEY }}
          KUBECTL_VERSION: "v1.22.0"
          IAM_VERSION: "0.5.4"
          KUBE_NAMESPACE: qa-email-ns
        with:
          args: rollout status deployment/qa-email-deployment -n $KUBE_NAMESPACE