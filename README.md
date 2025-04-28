# Factoring Marketplace MVP

A trucking and logistics financing marketplace built on Odoo Community Edition.

## Overview

This project implements a marketplace platform that connects carriers seeking funding with factoring companies. The platform is built using Odoo Community Edition, leveraging its built-in features while adding custom functionality specific to the factoring industry.

## Features

- Carrier registration and profile management
- Funding request submission and tracking
- Proposal management for factoring companies
- Document vault for secure file storage
- Real-time notifications and updates
- Mobile-responsive interface

## Technical Stack

- **Platform**: Odoo Community Edition
- **Frontend**: Odoo Website/Portal
- **Backend**: Odoo CRM, Attachments, Mail
- **Database**: PostgreSQL
- **Hosting**: Odoo.sh, AWS EC2, or similar

## Documentation

- [MVP Specification](factoring_marketplace_mvp_spec.md)
- [Technical Implementation](factoring_marketplace_technical_spec.md)
- [Wireframes](factoring_marketplace_wireframes.md)

## Getting Started

1. Clone the repository
2. Install Odoo Community Edition
3. Install the factoring_marketplace module
4. Configure the database and users
5. Access the platform through the web interface

## Development

### Prerequisites

- Python 3.7+
- PostgreSQL 12+
- Odoo 15.0+

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/factoring-marketplace.git

# Install dependencies
pip install -r requirements.txt

# Initialize the database
./odoo-bin -c odoo.conf -d factoring_marketplace -i factoring_marketplace
```

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the AGPL-3.0 License - see the [LICENSE](LICENSE) file for details.

## Contact

For questions or support, please contact the development team. 