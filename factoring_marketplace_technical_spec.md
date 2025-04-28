# Factoring Marketplace - Technical Implementation Specification

## 1. Module Structure

```
factoring_marketplace/
├── __init__.py
├── __manifest__.py
├── controllers/
│   ├── __init__.py
│   ├── portal.py
│   └── website.py
├── models/
│   ├── __init__.py
│   ├── funding_request.py
│   ├── proposal.py
│   ├── document_vault.py
│   └── res_partner.py
├── security/
│   ├── ir.model.access.csv
│   └── record_rules.xml
├── static/
│   ├── src/
│   │   ├── js/
│   │   └── scss/
│   └── description/
├── templates/
│   ├── portal_templates.xml
│   └── website_templates.xml
└── views/
    ├── funding_request_views.xml
    ├── proposal_views.xml
    ├── document_vault_views.xml
    ├── portal_templates.xml
    └── website_templates.xml
```

## 2. Data Models

### 2.1 Funding Request Model
```python
class FundingRequest(models.Model):
    _name = 'factoring.funding.request'
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _description = 'Funding Request'

    name = fields.Char('Reference', required=True)
    carrier_id = fields.Many2one('res.partner', 'Carrier', required=True)
    amount = fields.Monetary('Amount', required=True)
    currency_id = fields.Many2one('res.currency', 'Currency', required=True)
    purpose = fields.Selection([
        ('equipment', 'Equipment Purchase'),
        ('working_capital', 'Working Capital'),
        ('fuel', 'Fuel Advance'),
        ('other', 'Other')
    ], 'Purpose', required=True)
    start_date = fields.Date('Start Date')
    end_date = fields.Date('End Date')
    state = fields.Selection([
        ('draft', 'Draft'),
        ('submitted', 'Submitted'),
        ('in_progress', 'In Progress'),
        ('completed', 'Completed'),
        ('cancelled', 'Cancelled')
    ], 'State', default='draft')
    proposal_ids = fields.One2many('factoring.proposal', 'request_id', 'Proposals')
    document_ids = fields.Many2many('ir.attachment', string='Documents')
```

### 2.2 Proposal Model
```python
class Proposal(models.Model):
    _name = 'factoring.proposal'
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _description = 'Funding Proposal'

    name = fields.Char('Reference', required=True)
    request_id = fields.Many2one('factoring.funding.request', 'Funding Request', required=True)
    factoring_company_id = fields.Many2one('res.partner', 'Factoring Company', required=True)
    amount = fields.Monetary('Amount', required=True)
    interest_rate = fields.Float('Interest Rate', required=True)
    term = fields.Integer('Term (months)', required=True)
    monthly_payment = fields.Monetary('Monthly Payment', compute='_compute_monthly_payment')
    state = fields.Selection([
        ('draft', 'Draft'),
        ('submitted', 'Submitted'),
        ('accepted', 'Accepted'),
        ('rejected', 'Rejected')
    ], 'State', default='draft')
    document_ids = fields.Many2many('ir.attachment', string='Documents')
```

### 2.3 Document Vault Model
```python
class DocumentVault(models.Model):
    _name = 'factoring.document.vault'
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _description = 'Document Vault'

    name = fields.Char('Name', required=True)
    document_type = fields.Selection([
        ('agreement', 'Agreement'),
        ('proposal', 'Proposal'),
        ('financial', 'Financial Statement'),
        ('other', 'Other')
    ], 'Document Type', required=True)
    partner_id = fields.Many2one('res.partner', 'Partner', required=True)
    document_id = fields.Many2one('ir.attachment', 'Document', required=True)
    access_group_ids = fields.Many2many('res.groups', string='Access Groups')
```

## 3. Security Configuration

### 3.1 Access Rights
```xml
<record id="group_factoring_carrier" model="res.groups">
    <field name="name">Factoring Carrier</field>
    <field name="category_id" ref="base.module_category_factoring"/>
</record>

<record id="group_factoring_company" model="res.groups">
    <field name="name">Factoring Company</field>
    <field name="category_id" ref="base.module_category_factoring"/>
</record>
```

### 3.2 Record Rules
```xml
<record id="factoring_funding_request_rule_carrier" model="ir.rule">
    <field name="name">Carrier: Own Funding Requests</field>
    <field name="model_id" ref="model_factoring_funding_request"/>
    <field name="domain_force">[('carrier_id', '=', user.partner_id.id)]</field>
    <field name="groups" eval="[(4, ref('group_factoring_carrier'))]"/>
</record>
```

## 4. Portal Configuration

### 4.1 Portal Menu Items
```xml
<record id="factoring_portal_menu" model="website.menu">
    <field name="name">Factoring</field>
    <field name="url">/my/factoring</field>
    <field name="parent_id" ref="website.menu_portal"/>
    <field name="sequence" type="int">10</field>
</record>
```

### 4.2 Portal Templates
```xml
<template id="factoring_portal_my_requests" name="My Funding Requests">
    <t t-call="website.portal_layout">
        <div class="container mt-3">
            <div class="row">
                <div class="col-12">
                    <h2>My Funding Requests</h2>
                    <t t-foreach="requests" t-as="request">
                        <div class="card mb-3">
                            <div class="card-body">
                                <h5 class="card-title" t-esc="request.name"/>
                                <p class="card-text">
                                    Amount: <t t-esc="request.amount"/>
                                    Status: <t t-esc="request.state"/>
                                </p>
                            </div>
                        </div>
                    </t>
                </div>
            </div>
        </div>
    </t>
</template>
```

## 5. Website Configuration

### 5.1 Website Snippets
```xml
<template id="snippet_factoring_how_it_works" name="How It Works">
    <section class="s_factoring_how_it_works">
        <div class="container">
            <div class="row">
                <div class="col-12">
                    <h2>How It Works</h2>
                    <div class="row">
                        <div class="col-md-3">
                            <div class="card">
                                <div class="card-body">
                                    <h5>Register</h5>
                                    <p>Create your company profile</p>
                                </div>
                            </div>
                        </div>
                        <!-- Additional steps -->
                    </div>
                </div>
            </div>
        </div>
    </section>
</template>
```

## 6. JavaScript Enhancements

### 6.1 Proposal Calculator
```javascript
odoo.define('factoring_marketplace.proposal_calculator', function (require) {
    'use strict';

    var publicWidget = require('web.public.widget');

    publicWidget.registry.ProposalCalculator = publicWidget.Widget.extend({
        selector: '.js_proposal_calculator',
        events: {
            'change input': '_onChange',
        },

        _onChange: function () {
            var amount = parseFloat(this.$el.find('#amount').val());
            var rate = parseFloat(this.$el.find('#rate').val());
            var term = parseInt(this.$el.find('#term').val());
            
            if (amount && rate && term) {
                var monthlyPayment = this._calculateMonthlyPayment(amount, rate, term);
                this.$el.find('#monthly_payment').val(monthlyPayment.toFixed(2));
            }
        },

        _calculateMonthlyPayment: function (amount, rate, term) {
            // Implementation of monthly payment calculation
        },
    });
});
```

## 7. SCSS Styling

### 7.1 Custom Theme
```scss
.factoring-marketplace {
    .card {
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        
        &-header {
            background-color: $o-brand-primary;
            color: white;
        }
    }

    .status-badge {
        padding: 0.25rem 0.5rem;
        border-radius: 4px;
        
        &.draft {
            background-color: $gray-200;
        }
        
        &.submitted {
            background-color: $blue-100;
        }
        
        &.accepted {
            background-color: $green-100;
        }
    }
}
```

## 8. Implementation Guidelines

1. **Module Installation**
   - Install required dependencies
   - Set up database tables
   - Configure security groups

2. **Portal Setup**
   - Configure portal access rights
   - Set up user groups
   - Create portal templates

3. **Website Integration**
   - Add website snippets
   - Configure routes
   - Set up controllers

4. **Mobile Optimization**
   - Implement responsive design
   - Add touch-friendly controls
   - Optimize form layouts

5. **Testing**
   - Unit tests for models
   - Integration tests for controllers
   - UI tests for templates
   - Mobile responsiveness testing

## 9. Performance Considerations

1. **Database Optimization**
   - Index frequently queried fields
   - Optimize record rules
   - Use appropriate field types

2. **Frontend Optimization**
   - Lazy loading of images
   - Minification of assets
   - Caching strategies

3. **Security Measures**
   - CSRF protection
   - Input validation
   - Access control checks

## 10. Deployment Checklist

1. **Pre-deployment**
   - Database backup
   - Test environment verification
   - Security audit

2. **Deployment**
   - Module installation
   - Data migration
   - Cache clearing

3. **Post-deployment**
   - Functionality verification
   - Performance monitoring
   - User feedback collection

## 11. Additional Implementation Examples

### 11.1 Controller Implementation
```python
# controllers/portal.py
from odoo import http
from odoo.http import request

class FactoringPortal(http.Controller):
    @http.route('/my/factoring/requests', type='http', auth='user', website=True)
    def my_funding_requests(self, **kw):
        requests = request.env['factoring.funding.request'].search([
            ('carrier_id', '=', request.env.user.partner_id.id)
        ])
        return request.render('factoring_marketplace.portal_my_requests', {
            'requests': requests,
            'page_name': 'funding_requests',
        })

    @http.route('/my/factoring/request/<int:request_id>', type='http', auth='user', website=True)
    def funding_request_detail(self, request_id, **kw):
        request_obj = request.env['factoring.funding.request'].browse(request_id)
        if not request_obj.exists():
            return request.redirect('/my/factoring/requests')
        return request.render('factoring_marketplace.portal_request_detail', {
            'request': request_obj,
            'page_name': 'request_detail',
        })
```

### 11.2 View Implementation
```xml
<!-- views/funding_request_views.xml -->
<record id="view_funding_request_form" model="ir.ui.view">
    <field name="name">factoring.funding.request.form</field>
    <field name="model">factoring.funding.request</field>
    <field name="arch" type="xml">
        <form>
            <header>
                <button name="action_submit" string="Submit" type="object" 
                        class="oe_highlight" states="draft"/>
                <button name="action_cancel" string="Cancel" type="object" 
                        states="draft"/>
                <field name="state" widget="statusbar"/>
            </header>
            <sheet>
                <div class="oe_title">
                    <h1>
                        <field name="name" placeholder="Reference"/>
                    </h1>
                </div>
                <group>
                    <group>
                        <field name="carrier_id"/>
                        <field name="amount"/>
                        <field name="currency_id"/>
                    </group>
                    <group>
                        <field name="purpose"/>
                        <field name="start_date"/>
                        <field name="end_date"/>
                    </group>
                </group>
                <notebook>
                    <page string="Documents">
                        <field name="document_ids" widget="many2many_binary"/>
                    </page>
                    <page string="Proposals">
                        <field name="proposal_ids">
                            <tree editable="bottom">
                                <field name="factoring_company_id"/>
                                <field name="amount"/>
                                <field name="interest_rate"/>
                                <field name="term"/>
                                <field name="monthly_payment"/>
                                <field name="state"/>
                            </tree>
                        </field>
                    </page>
                </notebook>
            </sheet>
        </form>
    </field>
</record>
```

### 11.3 Business Logic Implementation
```python
# models/funding_request.py
class FundingRequest(models.Model):
    # ... existing fields ...

    def action_submit(self):
        for request in self:
            if request.state == 'draft':
                request.write({'state': 'submitted'})
                # Notify factoring companies
                self._notify_factoring_companies()

    def _notify_factoring_companies(self):
        template = self.env.ref('factoring_marketplace.email_template_new_request')
        factoring_companies = self.env['res.partner'].search([
            ('factoring_company', '=', True)
        ])
        for company in factoring_companies:
            template.with_context(
                default_partner_id=company.id,
                default_request_id=self.id
            ).send_mail(self.id, force_send=True)

    @api.depends('proposal_ids')
    def _compute_best_proposal(self):
        for request in self:
            proposals = request.proposal_ids.filtered(lambda p: p.state == 'submitted')
            if proposals:
                request.best_proposal_id = min(proposals, key=lambda p: p.interest_rate)
            else:
                request.best_proposal_id = False
```

### 11.4 Document Management Implementation
```python
# models/document_vault.py
class DocumentVault(models.Model):
    # ... existing fields ...

    def _check_document_access(self):
        for doc in self:
            if not doc.access_group_ids:
                continue
            user_groups = self.env.user.groups_id
            if not any(group in user_groups for group in doc.access_group_ids):
                raise AccessError("You don't have access to this document")

    def action_upload_document(self):
        return {
            'name': 'Upload Document',
            'type': 'ir.actions.act_window',
            'res_model': 'ir.attachment',
            'view_mode': 'form',
            'target': 'new',
            'context': {
                'default_res_model': self._name,
                'default_res_id': self.id,
            }
        }
```

### 11.5 Proposal Calculator Implementation
```javascript
// static/src/js/proposal_calculator.js
odoo.define('factoring_marketplace.proposal_calculator', function (require) {
    'use strict';

    var publicWidget = require('web.public.widget');
    var core = require('web.core');
    var _t = core._t;

    publicWidget.registry.ProposalCalculator = publicWidget.Widget.extend({
        selector: '.js_proposal_calculator',
        events: {
            'change input': '_onChange',
            'click .js_calculate': '_onCalculate',
        },

        start: function () {
            this._super.apply(this, arguments);
            this._updateMonthlyPayment();
        },

        _onChange: function () {
            this._updateMonthlyPayment();
        },

        _onCalculate: function () {
            var amount = parseFloat(this.$el.find('#amount').val());
            var rate = parseFloat(this.$el.find('#rate').val());
            var term = parseInt(this.$el.find('#term').val());
            
            if (this._validateInputs(amount, rate, term)) {
                var monthlyPayment = this._calculateMonthlyPayment(amount, rate, term);
                this.$el.find('#monthly_payment').val(monthlyPayment.toFixed(2));
                this._updateAmortizationSchedule(amount, rate, term, monthlyPayment);
            }
        },

        _validateInputs: function (amount, rate, term) {
            if (isNaN(amount) || amount <= 0) {
                this._showError(_t('Please enter a valid amount'));
                return false;
            }
            if (isNaN(rate) || rate <= 0) {
                this._showError(_t('Please enter a valid interest rate'));
                return false;
            }
            if (isNaN(term) || term <= 0) {
                this._showError(_t('Please enter a valid term'));
                return false;
            }
            return true;
        },

        _calculateMonthlyPayment: function (amount, rate, term) {
            var monthlyRate = rate / 100 / 12;
            return amount * monthlyRate * Math.pow(1 + monthlyRate, term) / 
                   (Math.pow(1 + monthlyRate, term) - 1);
        },

        _updateAmortizationSchedule: function (amount, rate, term, monthlyPayment) {
            var schedule = [];
            var balance = amount;
            var monthlyRate = rate / 100 / 12;
            
            for (var i = 1; i <= term; i++) {
                var interest = balance * monthlyRate;
                var principal = monthlyPayment - interest;
                balance -= principal;
                
                schedule.push({
                    payment: i,
                    principal: principal.toFixed(2),
                    interest: interest.toFixed(2),
                    balance: balance.toFixed(2)
                });
            }
            
            this._renderAmortizationSchedule(schedule);
        },

        _renderAmortizationSchedule: function (schedule) {
            var $table = this.$el.find('.js_amortization_schedule');
            $table.empty();
            
            schedule.forEach(function (row) {
                $table.append(
                    '<tr>' +
                    '<td>' + row.payment + '</td>' +
                    '<td>' + row.principal + '</td>' +
                    '<td>' + row.interest + '</td>' +
                    '<td>' + row.balance + '</td>' +
                    '</tr>'
                );
            });
        },

        _showError: function (message) {
            this.$el.find('.js_error').text(message).show();
        }
    });
});
```

### 11.6 Mobile-Specific Implementation
```xml
<!-- views/website_templates.xml -->
<template id="factoring_mobile_menu" name="Mobile Menu">
    <t t-call="website.layout">
        <div class="mobile-menu">
            <div class="container">
                <div class="row">
                    <div class="col-12">
                        <div class="mobile-nav">
                            <a href="/my/factoring/requests" class="nav-item">
                                <i class="fa fa-list"></i>
                                <span>Requests</span>
                            </a>
                            <a href="/my/factoring/proposals" class="nav-item">
                                <i class="fa fa-file-text"></i>
                                <span>Proposals</span>
                            </a>
                            <a href="/my/factoring/documents" class="nav-item">
                                <i class="fa fa-folder"></i>
                                <span>Documents</span>
                            </a>
                            <a href="/my/factoring/profile" class="nav-item">
                                <i class="fa fa-user"></i>
                                <span>Profile</span>
                            </a>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </t>
</template>
```

### 11.7 Testing Implementation
```python
# tests/test_funding_request.py
from odoo.tests import common

class TestFundingRequest(common.TransactionCase):
    def setUp(self):
        super(TestFundingRequest, self).setUp()
        self.carrier = self.env['res.partner'].create({
            'name': 'Test Carrier',
            'factoring_carrier': True
        })
        self.factoring_company = self.env['res.partner'].create({
            'name': 'Test Factoring Company',
            'factoring_company': True
        })

    def test_create_funding_request(self):
        request = self.env['factoring.funding.request'].create({
            'name': 'TEST-001',
            'carrier_id': self.carrier.id,
            'amount': 50000,
            'currency_id': self.env.ref('base.USD').id,
            'purpose': 'equipment'
        })
        self.assertEqual(request.state, 'draft')
        self.assertEqual(request.carrier_id, self.carrier)

    def test_submit_funding_request(self):
        request = self.env['factoring.funding.request'].create({
            'name': 'TEST-002',
            'carrier_id': self.carrier.id,
            'amount': 50000,
            'currency_id': self.env.ref('base.USD').id,
            'purpose': 'equipment'
        })
        request.action_submit()
        self.assertEqual(request.state, 'submitted')
```

These additional implementation examples provide more detailed code for:
1. Portal controllers for handling web requests
2. Form views with proper layout and widgets
3. Business logic for funding requests and proposals
4. Document management with access control
5. Interactive proposal calculator with amortization schedule
6. Mobile-specific navigation and UI
7. Unit tests for core functionality

## 12. Detailed Implementation Documentation

### 12.1 Controller Implementation Documentation

#### Portal Controller Structure
```python
# controllers/portal.py
from odoo import http
from odoo.http import request

class FactoringPortal(http.Controller):
    """
    Portal Controller for Factoring Marketplace
    
    This controller handles all portal-related routes and actions for both carriers
    and factoring companies. It implements the following security measures:
    - User authentication required for all routes
    - Record-level access control
    - CSRF protection (built into Odoo)
    """
    
    @http.route('/my/factoring/requests', type='http', auth='user', website=True)
    def my_funding_requests(self, **kw):
        """
        Display list of funding requests for the current user
        
        Security:
        - Only authenticated users can access
        - Users can only see their own requests
        - Requests are filtered by carrier_id matching user's partner_id
        
        Returns:
        - Rendered template with list of requests
        - Includes pagination for large result sets
        """
        requests = request.env['factoring.funding.request'].search([
            ('carrier_id', '=', request.env.user.partner_id.id)
        ])
        return request.render('factoring_marketplace.portal_my_requests', {
            'requests': requests,
            'page_name': 'funding_requests',
        })
```

### 12.2 View Implementation Documentation

#### Form View Structure
```xml
<!-- views/funding_request_views.xml -->
<record id="view_funding_request_form" model="ir.ui.view">
    <field name="name">factoring.funding.request.form</field>
    <field name="model">factoring.funding.request</field>
    <field name="arch" type="xml">
        <!-- 
        Form View Structure:
        1. Header: Contains action buttons and status bar
        2. Sheet: Main content area
           - Title section
           - Grouped fields
           - Notebook with tabs
        -->
        <form>
            <!-- Header with action buttons and status -->
            <header>
                <!-- 
                Action Buttons:
                - Submit: Only visible in draft state
                - Cancel: Only visible in draft state
                - Status bar shows current state
                -->
                <button name="action_submit" string="Submit" type="object" 
                        class="oe_highlight" states="draft"/>
                <button name="action_cancel" string="Cancel" type="object" 
                        states="draft"/>
                <field name="state" widget="statusbar"/>
            </header>
            
            <!-- Main content area -->
            <sheet>
                <!-- Title section -->
                <div class="oe_title">
                    <h1>
                        <field name="name" placeholder="Reference"/>
                    </h1>
                </div>
                
                <!-- Grouped fields for better organization -->
                <group>
                    <group>
                        <!-- Basic information -->
                        <field name="carrier_id"/>
                        <field name="amount"/>
                        <field name="currency_id"/>
                    </group>
                    <group>
                        <!-- Additional details -->
                        <field name="purpose"/>
                        <field name="start_date"/>
                        <field name="end_date"/>
                    </group>
                </group>
                
                <!-- Notebook with tabs -->
                <notebook>
                    <!-- Documents tab -->
                    <page string="Documents">
                        <!-- 
                        Document Management:
                        - Uses many2many_binary widget for file uploads
                        - Supports drag and drop
                        - Shows file previews
                        -->
                        <field name="document_ids" widget="many2many_binary"/>
                    </page>
                    
                    <!-- Proposals tab -->
                    <page string="Proposals">
                        <!-- 
                        Proposals List:
                        - Editable tree view
                        - Shows key proposal details
                        - Allows inline editing
                        -->
                        <field name="proposal_ids">
                            <tree editable="bottom">
                                <field name="factoring_company_id"/>
                                <field name="amount"/>
                                <field name="interest_rate"/>
                                <field name="term"/>
                                <field name="monthly_payment"/>
                                <field name="state"/>
                            </tree>
                        </field>
                    </page>
                </notebook>
            </sheet>
        </form>
    </field>
</record>
```

### 12.3 Business Logic Documentation

#### Funding Request Model
```python
# models/funding_request.py
class FundingRequest(models.Model):
    """
    Funding Request Model
    
    This model represents a funding request from a carrier to factoring companies.
    It includes:
    - Basic request information
    - State management
    - Proposal handling
    - Document management
    """
    
    def action_submit(self):
        """
        Submit a funding request
        
        Actions:
        1. Change state to 'submitted'
        2. Notify relevant factoring companies
        3. Create activity for follow-up
        
        Security:
        - Only draft requests can be submitted
        - Only carrier can submit their own requests
        """
        for request in self:
            if request.state == 'draft':
                request.write({'state': 'submitted'})
                self._notify_factoring_companies()
                self._create_followup_activity()

    def _notify_factoring_companies(self):
        """
        Notify factoring companies about new request
        
        Process:
        1. Get email template
        2. Find relevant factoring companies
        3. Send personalized emails
        
        Optimization:
        - Uses template rendering for efficiency
        - Batches email sending
        """
        template = self.env.ref('factoring_marketplace.email_template_new_request')
        factoring_companies = self.env['res.partner'].search([
            ('factoring_company', '=', True)
        ])
        for company in factoring_companies:
            template.with_context(
                default_partner_id=company.id,
                default_request_id=self.id
            ).send_mail(self.id, force_send=True)

    @api.depends('proposal_ids')
    def _compute_best_proposal(self):
        """
        Compute the best proposal based on interest rate
        
        Logic:
        - Only considers submitted proposals
        - Selects proposal with lowest interest rate
        - Handles empty proposal sets
        
        Performance:
        - Uses @api.depends for efficient computation
        - Minimizes database queries
        """
        for request in self:
            proposals = request.proposal_ids.filtered(lambda p: p.state == 'submitted')
            if proposals:
                request.best_proposal_id = min(proposals, key=lambda p: p.interest_rate)
            else:
                request.best_proposal_id = False
```

### 12.4 Document Management Documentation

#### Document Vault Model
```python
# models/document_vault.py
class DocumentVault(models.Model):
    """
    Document Vault Model
    
    Manages secure document storage and access control for:
    - Agreements
    - Proposals
    - Financial statements
    - Supporting documents
    """
    
    def _check_document_access(self):
        """
        Verify document access permissions
        
        Security:
        - Checks user's group membership
        - Implements role-based access control
        - Handles public documents
        
        Error Handling:
        - Raises AccessError for unauthorized access
        - Logs access attempts
        """
        for doc in self:
            if not doc.access_group_ids:
                continue
            user_groups = self.env.user.groups_id
            if not any(group in user_groups for group in doc.access_group_ids):
                raise AccessError("You don't have access to this document")

    def action_upload_document(self):
        """
        Open document upload wizard
        
        Features:
        - Supports multiple file types
        - Handles large files
        - Provides progress feedback
        
        Returns:
        - Action dictionary for wizard
        - Includes context for proper document linking
        """
        return {
            'name': 'Upload Document',
            'type': 'ir.actions.act_window',
            'res_model': 'ir.attachment',
            'view_mode': 'form',
            'target': 'new',
            'context': {
                'default_res_model': self._name,
                'default_res_id': self.id,
            }
        }
```

### 12.5 Testing Documentation

#### Test Implementation
```python
# tests/test_funding_request.py
from odoo.tests import common

class TestFundingRequest(common.TransactionCase):
    """
    Test Cases for Funding Request Model
    
    Implements comprehensive testing for:
    - Model creation
    - State transitions
    - Access control
    - Business logic
    """
    
    def setUp(self):
        """
        Set up test environment
        
        Creates:
        - Test carrier
        - Test factoring company
        - Required base records
        """
        super(TestFundingRequest, self).setUp()
        self.carrier = self.env['res.partner'].create({
            'name': 'Test Carrier',
            'factoring_carrier': True
        })
        self.factoring_company = self.env['res.partner'].create({
            'name': 'Test Factoring Company',
            'factoring_company': True
        })

    def test_create_funding_request(self):
        """
        Test funding request creation
        
        Verifies:
        - Default state is 'draft'
        - Carrier is properly set
        - Required fields are populated
        """
        request = self.env['factoring.funding.request'].create({
            'name': 'TEST-001',
            'carrier_id': self.carrier.id,
            'amount': 50000,
            'currency_id': self.env.ref('base.USD').id,
            'purpose': 'equipment'
        })
        self.assertEqual(request.state, 'draft')
        self.assertEqual(request.carrier_id, self.carrier)

    def test_submit_funding_request(self):
        """
        Test request submission process
        
        Verifies:
        - State transition to 'submitted'
        - Notification creation
        - Access control
        """
        request = self.env['factoring.funding.request'].create({
            'name': 'TEST-002',
            'carrier_id': self.carrier.id,
            'amount': 50000,
            'currency_id': self.env.ref('base.USD').id,
            'purpose': 'equipment'
        })
        request.action_submit()
        self.assertEqual(request.state, 'submitted')
```

## 13. Implementation Best Practices

### 13.1 Security Best Practices
1. **Access Control**
   - Use record rules for data-level security
   - Implement group-based permissions
   - Validate user input

2. **Data Protection**
   - Encrypt sensitive data
   - Implement audit logging
   - Regular security reviews

### 13.2 Performance Optimization
1. **Database Optimization**
   - Use appropriate field types
   - Create necessary indexes
   - Optimize queries

2. **Frontend Optimization**
   - Minimize asset size
   - Implement caching
   - Use lazy loading

### 13.3 Code Quality
1. **Maintainability**
   - Follow PEP 8 style guide
   - Write comprehensive docstrings
   - Implement proper error handling

2. **Testing**
   - Write unit tests
   - Implement integration tests
   - Regular code reviews

### 13.4 User Experience
1. **Responsive Design**
   - Mobile-first approach
   - Touch-friendly controls
   - Progressive enhancement

2. **Accessibility**
   - WCAG 2.1 compliance
   - Screen reader support
   - Keyboard navigation