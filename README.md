# Factoring Marketplace Platform

A multi-tenant factoring marketplace connecting Carriers with Factoring Companies.

## Technology Stack
- Frontend: React + TypeScript
- Backend: C# AWS Lambda
- Database: PostgreSQL RDS
- Authentication: AWS Cognito
- Infrastructure: AWS (CloudFront, API Gateway, Lambda, SNS, S3)

## Project Structure
```
project-root/
├── .github/
│   └── workflows/                 # CI/CD workflows
├── src/
│   ├── Frontend/                  # React application
│   │   ├── components/
│   │   ├── pages/
│   │   └── services/
│   └── Backend/                   # C# Lambda functions
│       ├── Common/                # Shared code
│       ├── Functions/             # Lambda handlers
│       └── Tests/                 # Unit tests
├── infrastructure/                # IaC (AWS CDK or Serverless Framework)
├── docs/                         
│   ├── architecture/
│   ├── api/
│   └── wireframes/
└── README.md
```

## Getting Started

### Prerequisites
- .NET 6.0+
- Node.js 16+
- AWS CLI configured
- Docker (for local development)

### Local Development
1. Clone the repository
2. Setup local development environment
   ```bash
   cd src/Frontend
   npm install
   
   cd ../Backend
   dotnet restore
   ```

3. Run the application
   ```bash
   # Frontend
   npm start
   
   # Backend
   dotnet run
   ```

## Testing
- Frontend: Jest + React Testing Library
- Backend: xUnit

## Deployment
Automated via GitHub Actions to AWS

## Documentation
- [Architecture](docs/architecture/README.md)
- [API Documentation](docs/api/README.md)
- [Wireframes](docs/wireframes/README.md)

## Contributing
Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct and the process for submitting pull requests.

## License
This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details 