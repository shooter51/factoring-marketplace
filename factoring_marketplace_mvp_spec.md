# Factoring Marketplace MVP Specification

## Platform: Odoo Community Edition (Frontend + Backend)
### Domain: Trucking and Logistics Financing

## Key Development Philosophy
- Use Built-in Odoo Features Whenever Possible
- Minimize Custom Development
- Configuration Over Customization
- Lean MVP for Fastest Release and Easier Maintenance

## Personas
- Carrier (seeks funding)
- Factoring Company (provides funding)

## Core Features Using Built-in Odoo Community Capabilities

| Feature | Approach |
|---------|----------|
| Authentication | Odoo's internal login system (Users & Groups) |
| User Profiles | Extend Odoo's res.partner model (Companies & Contacts) |
| Company Registration | Portal Registration Forms |
| Funding Request Tracking | Customized CRM Lead or Opportunity object |
| Proposal Submission | CRM stages/custom fields on Opportunities |
| Document Upload | Odoo Attachments (basic document handling) |
| Notifications | Odoo Email templates + Chatter Messaging |
| Dashboards | CRM Kanban and List Views, customized |
| Authorization (Role Management) | Odoo Groups (Carrier, Factoring Company) |

## Carrier Journey
1. Register account (Portal / Website)
2. Create Company Profile
3. Submit Funding Request (CRM Opportunity)
4. View/manage incoming Proposals
5. Accept or decline Proposals
6. Access finalized agreements via attachments

## Factoring Company Journey
1. Register account (Portal / Website)
2. Create Company Profile
3. View Funding Requests
4. Submit Proposals (custom fields)
5. Upload Proposal Documents
6. Get notified upon Carrier acceptance

## MVP Dashboards

### Carriers:
- Active Funding Requests
- Proposals Received
- Past Fundings
- Recent Activity

### Factoring Companies:
- New Opportunities
- Submitted Proposals
- Financial Stats
- Recent Activity

## Technical Architecture Overview
- Frontend: Odoo Website/Portal
- Backend: Odoo CRM, Attachments, Mail
- Database: PostgreSQL
- Hosting: Odoo.sh, AWS EC2, or similar

### Security:
- Record Rules/Access Rights Management
- Encryption at Database Level (if hosted on cloud services)
- Notifications: Email and Chatter updates

## Limitations of Odoo Community and Mitigations

| Missing Feature | Impact | Solution |
|----------------|--------|----------|
| Odoo Sign (E-Signatures) | Not available | Integrate third-party e-signature service (e.g., DocuSign) or develop custom module |
| Advanced Document Management | Limited | Use Attachments for basic needs, custom Document Vault module for access control |
| Odoo Studio (App Builder) | Not available | Manual development of custom modules |

## Light Custom Modules Required

| Module Name | Purpose |
|------------|---------|
| Funding Requests | Specialized fields/forms for Carrier funding needs |
| Proposal Management | Factoring Companies submit offers and track status |
| Document Vault | Secure access to signed agreements for Carrier and Factoring Company |

## Future Enhancements (Post-MVP)
- Real-time Push Notifications
- Carrier Credit Scoring
- AI-based Proposal Matching
- Direct Payment Integrations
- Full Mobile App Experience

## Summary
Odoo Community Edition provides a strong foundation for building the Factoring Marketplace MVP with minimal custom development. All critical workflows are achievable using built-in modules, while light extensions will handle special business requirements like secure document vaulting and third-party electronic signature integrations. 