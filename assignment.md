# Assignment (Optional)

## Brief

Implement a complete CI/CD pipeline with automated deployment to Railway for your Spring Boot application.

1. **Set Up Railway and Deploy Manually**
   - Create a Railway account at https://railway.app (sign up with GitHub)
   - Add your credit card (required for free tier, $5/month credit)
   - Create a new Railway project by deploying from GitHub:
     - Select "Deploy from GitHub repo"
     - Choose your devops-demo repository
     - Railway will detect your Dockerfile and build automatically
   - Claim the project to make it permanent
   - Generate a public domain for your service:
     - Go to Settings → Networking
     - Click "Generate Domain"
     - Set port to 8080
   - Test your application by accessing: `https://your-railway-url.up.railway.app/hello`
   - Take a screenshot showing your application running on Railway

2. **Configure Automated Deployment with CircleCI**
   - Get Railway credentials:
     - API Token: Account Settings → Tokens → Create Token
     - Project ID: Project Settings → Copy Project ID
     - Service ID: Service Settings → Copy Service ID
   - Add environment variables to CircleCI:
     - Go to Project Settings → Environment Variables
     - Add `RAILWAY_TOKEN` with your API token
     - Add `RAILWAY_PROJECT_ID` with your project ID
     - Add `RAILWAY_SERVICE_ID` with your service ID
   - Update your `.circleci/config.yml` to add a deploy job:
     - Use `cimg/base:stable` Docker image
     - Install Railway CLI with: `curl -fsSL https://railway.app/install.sh | sh`
     - Set PATH to include Railway CLI: `export PATH="/root/.railway/bin:$PATH"`
     - Login with: `railway login --browserless`
     - Link to project: `railway link $RAILWAY_PROJECT_ID`
     - Trigger deployment: `railway up --service $RAILWAY_SERVICE_ID`
   - Update your workflow to run deploy after publish succeeds
   - Commit and push your updated config.yml
   - Watch the complete pipeline run: build → test → publish → deploy
   - Verify all four jobs pass in CircleCI
   - Check Railway dashboard to see the deployment triggered

3. **Test the Full CI/CD Pipeline**
   - Make a small change to your DemoController (update the /hello message)
   - Update your test to match the new message
   - Commit and push the change
   - Watch the automated pipeline:
     - CircleCI builds, tests, publishes Docker image
     - CircleCI triggers Railway deployment
     - Railway pulls latest code and redeploys
   - Verify the change appears on your live Railway URL
   - Take screenshots showing:
     - CircleCI pipeline with all 4 jobs passing
     - Railway deployment logs
     - Your updated message live on Railway
   - Write a brief report (4-5 sentences) explaining:
     - The complete flow from code push to production
     - How long the deployment took
     - What would happen if your tests failed
     - The benefits of automated deployment vs manual deployment

## Submission (Optional)

- Submit the URL of the GitHub Repository that contains your work to NTU black board.
- Should you reference the work of your classmate(s) or online resources, give them credit by adding either the name of your classmate or URL.

## References
- Java: https://docs.oracle.com/javase/
- Spring Boot: https://docs.spring.io/spring-boot/docs/current/reference/html/
- PostgreSQL: https://www.postgresql.org/docs/
- OWASP: https://cheatsheetseries.owasp.org/