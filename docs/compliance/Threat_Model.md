# Threat Model - Support Console

## Overview

### Purpose
The Support Console is a Rails-based web application that provides two primary functions:
1. **Visitor Newsletter Signup**: Allows website visitors to subscribe to a newsletter via email integration with MailChimp
2. **Contact Form**: Enables users to send contact messages to the website owner via email

### Scope
This threat model covers the entire Support Console application, including:
- Web application endpoints and controllers
- Email integration with MailChimp API
- Contact form email delivery system
- Data validation and processing
- User input handling and storage

### Technology Stack
- **Framework**: Ruby on Rails 5.1.2
- **Language**: Ruby 2.4.1
- **Database**: SQLite3 (development/test), PostgreSQL (production)
- **Web Server**: Puma
- **External Integrations**: MailChimp API (Gibbon gem), SMTP email delivery

## Data Flow Diagram

```
┌─────────────┐
│   Visitor   │
└──────┬──────┘
       │
       │ HTTPS
       ▼
┌─────────────────────────────────────────────┐
│         Support Console Application         │
│  ┌────────────────┐    ┌─────────────────┐ │
│  │   Visitors     │    │    Contacts     │ │
│  │  Controller    │    │   Controller    │ │
│  └────────┬───────┘    └────────┬────────┘ │
│           │                     │           │
│           ▼                     ▼           │
│  ┌────────────────┐    ┌─────────────────┐ │
│  │    Visitor     │    │     Contact     │ │
│  │     Model      │    │      Model      │ │
│  └────────┬───────┘    └────────┬────────┘ │
└───────────┼────────────────────┼───────────┘
            │                    │
            │                    │
            ▼                    ▼
    ┌───────────────┐    ┌──────────────┐
    │   MailChimp   │    │ SMTP Server  │
    │      API      │    │  (Email)     │
    └───────────────┘    └──────────────┘
```

## Dependencies

### External Libraries
- **rails** (~> 5.1.2) - Web application framework
- **puma** (~> 3.7) - Web server
- **gibbon** - MailChimp API integration
- **bootstrap-sass** - Frontend CSS framework
- **jquery-rails** - JavaScript library
- **sass-rails** (~> 5.0) - Asset pipeline
- **uglifier** (>= 1.3.0) - JavaScript compressor
- **coffee-rails** (~> 4.2) - CoffeeScript support
- **turbolinks** (~> 5) - Page navigation optimization
- **jbuilder** (~> 2.5) - JSON API builder
- **high_voltage** - Static page management
- **sqlite3** (development/test) - Database adapter
- **pg** (production) - PostgreSQL adapter

### External Services
- **MailChimp API** - Newsletter subscription management
- **SMTP Server** - Email delivery for contact messages
- **DNS** - Domain name resolution
- **TLS/SSL Certificate Authority** - HTTPS encryption

### Infrastructure Dependencies
- **Ruby Runtime** (2.4.1)
- **Web Server** (Puma)
- **Database Server** (SQLite3/PostgreSQL)
- **Operating System** - Host OS and dependencies

## Entry Points

### EP1: Root Page (GET /)
- **Description**: Landing page displaying visitor signup form
- **Route**: `GET /`
- **Controller**: `VisitorsController#new`
- **Access**: Public, unauthenticated
- **Input**: None (initial page load)
- **Trust Level**: Anonymous

### EP2: Visitor Signup Form (POST /visitors)
- **Description**: Processes visitor email subscription
- **Route**: `POST /visitors`
- **Controller**: `VisitorsController#create`
- **Access**: Public, unauthenticated
- **Input**: Email address (form parameter)
- **Trust Level**: Anonymous

### EP3: Contact Form Page (GET /contacts/new)
- **Description**: Displays contact form
- **Route**: `GET /contacts/new`
- **Controller**: `ContactsController#new`
- **Access**: Public, unauthenticated
- **Input**: None (initial page load)
- **Trust Level**: Anonymous

### EP4: Contact Form Submission (POST /contacts)
- **Description**: Processes contact message submission
- **Route**: `POST /contacts`
- **Controller**: `ContactsController#create`
- **Access**: Public, unauthenticated
- **Input**: Name, email, message content (form parameters)
- **Trust Level**: Anonymous

### EP5: Static Assets
- **Description**: CSS, JavaScript, and image files
- **Access**: Public, unauthenticated
- **Input**: Asset requests via HTTP GET
- **Trust Level**: Anonymous

## Exit Points

### EX1: MailChimp API Integration
- **Description**: Subscriber email data sent to MailChimp
- **Protocol**: HTTPS
- **Endpoint**: MailChimp API (lists.members.create)
- **Data**: Email address, subscription status
- **Authentication**: API key

### EX2: SMTP Email Delivery
- **Description**: Contact form messages sent via email
- **Protocol**: SMTP (typically port 25, 465, or 587)
- **Destination**: Configured SMTP server
- **Data**: Contact name, email, message content
- **Authentication**: SMTP credentials (if configured)

### EX3: Application Logs
- **Description**: Application events and errors logged to files
- **Location**: `log/` directory
- **Data**: Request parameters, error messages, MailChimp subscription events
- **Access**: Server file system

### EX4: HTTP Responses
- **Description**: HTML pages, JSON responses, redirects, and error messages
- **Protocol**: HTTP/HTTPS
- **Data**: Form validation errors, success messages, rendered views
- **Audience**: End users

## Assets

### AS1: User Personal Information
- **Description**: Email addresses, names, and message content
- **Sensitivity**: Medium-High
- **Storage**: Transient (not persisted in database, sent to external services)
- **Regulations**: GDPR, CAN-SPAM, data privacy laws

### AS2: MailChimp API Credentials
- **Description**: API key and list ID for MailChimp integration
- **Sensitivity**: High
- **Storage**: Rails secrets configuration
- **Impact**: Unauthorized access could allow spam, data theft, or account compromise

### AS3: SMTP Credentials
- **Description**: Email server authentication credentials
- **Sensitivity**: High
- **Storage**: Rails configuration files
- **Impact**: Unauthorized access could enable email spoofing or spam

### AS4: Owner Email Address
- **Description**: Email address where contact messages are sent
- **Sensitivity**: Low-Medium
- **Storage**: Rails secrets configuration
- **Impact**: Exposure could lead to targeted spam

### AS5: Application Source Code
- **Description**: Ruby on Rails application code
- **Sensitivity**: Medium
- **Storage**: Source code repository
- **Impact**: Exposure could reveal business logic and vulnerabilities

### AS6: Session Data
- **Description**: Rails session cookies and flash messages
- **Sensitivity**: Low
- **Storage**: Client-side cookies (encrypted)
- **Impact**: Session hijacking could lead to CSRF attacks

### AS7: Application Logs
- **Description**: Server logs containing requests and errors
- **Sensitivity**: Medium
- **Storage**: File system (`log/` directory)
- **Impact**: May contain sensitive information like email addresses

## Trust Levels

### TL1: Anonymous User
- **Description**: Unauthenticated website visitor
- **Access**: Can view public pages, submit visitor signup form, submit contact form
- **Restrictions**: Rate limiting (if implemented), input validation
- **Risk Level**: Highest - untrusted input source

### TL2: Application Server
- **Description**: The Rails application runtime environment
- **Access**: Full access to application code, configuration, file system
- **Restrictions**: OS-level permissions, security policies
- **Risk Level**: Medium - trusted but must be properly configured

### TL3: Database Server
- **Description**: SQLite/PostgreSQL database instance
- **Access**: Data storage and retrieval within application scope
- **Restrictions**: Connection authentication, network isolation
- **Risk Level**: Low - internal trusted component

### TL4: External Services (MailChimp, SMTP)
- **Description**: Third-party API services
- **Access**: Receive user-provided data via authenticated API calls
- **Restrictions**: API authentication, TLS encryption
- **Risk Level**: Medium - trusted partners but external attack surface

### TL5: System Administrator
- **Description**: Personnel with server and application access
- **Access**: Full system access, configuration management, deployment
- **Restrictions**: Access controls, audit logging, least privilege
- **Risk Level**: Low - highly trusted but potential insider threat

## STRIDE Threat Analysis

### Spoofing

#### S1: Email Address Spoofing in Contact Form
- **Threat**: Attacker submits contact form with spoofed sender email address
- **Impact**: Recipient may trust fraudulent email, potential for phishing
- **Affected Components**: ContactsController, UserMailer
- **Likelihood**: High
- **Severity**: Medium

#### S2: MailChimp API Key Compromise
- **Threat**: Attacker obtains MailChimp API credentials from configuration
- **Impact**: Unauthorized access to subscriber list, potential data breach
- **Affected Components**: Rails secrets, Visitor model
- **Likelihood**: Low
- **Severity**: High

#### S3: SMTP Credential Theft
- **Threat**: SMTP credentials exposed in configuration files or environment
- **Impact**: Email account compromise, spam distribution
- **Affected Components**: Rails mailer configuration
- **Likelihood**: Low
- **Severity**: High

### Tampering

#### T1: Form Parameter Manipulation
- **Threat**: Attacker modifies POST parameters to inject malicious data
- **Impact**: XSS, code injection, data corruption
- **Affected Components**: Controllers (strong parameters), models (validation)
- **Likelihood**: Medium
- **Severity**: High

#### T2: Session Cookie Tampering
- **Threat**: Attacker attempts to modify encrypted session cookies
- **Impact**: Session fixation, unauthorized access
- **Affected Components**: Rails session management
- **Likelihood**: Low
- **Severity**: Medium

#### T3: Configuration File Modification
- **Threat**: Unauthorized modification of secrets.yml or other config files
- **Impact**: System compromise, credential theft, service disruption
- **Affected Components**: Configuration management, deployment process
- **Likelihood**: Low
- **Severity**: Critical

#### T4: Log File Tampering
- **Threat**: Attacker modifies or deletes log files to hide malicious activity
- **Impact**: Loss of audit trail, inability to detect intrusions
- **Affected Components**: File system permissions, logging infrastructure
- **Likelihood**: Low
- **Severity**: Medium

### Repudiation

#### R1: Contact Form Submission Denial
- **Threat**: User denies sending a contact message
- **Impact**: Disputes over communication, lack of accountability
- **Affected Components**: Logging, email headers
- **Likelihood**: Medium
- **Severity**: Low

#### R2: Newsletter Subscription Denial
- **Threat**: User claims they did not subscribe to newsletter
- **Impact**: CAN-SPAM compliance issues, reputation damage
- **Affected Components**: MailChimp integration, logging
- **Likelihood**: Medium
- **Severity**: Medium

#### R3: Insufficient Audit Logging
- **Threat**: Lack of detailed logs for security events
- **Impact**: Cannot track unauthorized access or investigate incidents
- **Affected Components**: Rails logging, security monitoring
- **Likelihood**: Medium
- **Severity**: Medium

### Information Disclosure

#### ID1: Sensitive Data in Logs
- **Threat**: Email addresses and other PII logged to files
- **Impact**: Privacy violation, GDPR non-compliance
- **Affected Components**: Rails logger, application logs
- **Likelihood**: High
- **Severity**: Medium

#### ID2: API Credentials in Source Code
- **Threat**: Hardcoded credentials or secrets committed to repository
- **Impact**: Credential compromise, unauthorized API access
- **Affected Components**: Version control, configuration management
- **Likelihood**: Medium
- **Severity**: High

#### ID3: Detailed Error Messages
- **Threat**: Stack traces and detailed errors exposed to users
- **Impact**: Information leakage about system internals
- **Affected Components**: Error handling, exception pages
- **Likelihood**: Medium
- **Severity**: Low

#### ID4: Insecure Direct Object References
- **Threat**: Predictable or exposed internal object identifiers
- **Impact**: Unauthorized data access (limited risk given no database storage)
- **Affected Components**: Controllers, routing
- **Likelihood**: Low
- **Severity**: Low

#### ID5: Missing Security Headers
- **Threat**: Lack of security headers (CSP, HSTS, X-Frame-Options)
- **Impact**: Vulnerability to XSS, clickjacking, MITM attacks
- **Affected Components**: HTTP response headers
- **Likelihood**: High
- **Severity**: Medium

### Denial of Service

#### DOS1: Form Submission Flood
- **Threat**: Attacker submits large volume of contact/signup forms
- **Impact**: Resource exhaustion, email flood, MailChimp API quota
- **Affected Components**: Controllers, MailChimp API, email delivery
- **Likelihood**: High
- **Severity**: Medium

#### DOS2: Large Payload Attacks
- **Threat**: Submitting extremely large message content
- **Impact**: Memory exhaustion, slow response times
- **Affected Components**: Parameter parsing, validation
- **Likelihood**: Medium
- **Severity**: Low

#### DOS3: Slowloris Attack
- **Threat**: Slow HTTP attacks to exhaust server connections
- **Impact**: Service unavailability
- **Affected Components**: Puma web server
- **Likelihood**: Medium
- **Severity**: Medium

#### DOS4: External Service Dependency Failure
- **Threat**: MailChimp or SMTP service outage affects application
- **Impact**: Failed subscriptions, error pages, poor user experience
- **Affected Components**: External API integrations
- **Likelihood**: Medium
- **Severity**: Low

### Elevation of Privilege

#### EOP1: SQL Injection
- **Threat**: Attacker injects SQL commands through form inputs
- **Impact**: Database compromise (limited - no direct DB queries in current code)
- **Affected Components**: Database queries (minimal risk)
- **Likelihood**: Low
- **Severity**: Critical (if exploitable)

#### EOP2: Command Injection
- **Threat**: Attacker injects OS commands via unsanitized input
- **Impact**: Remote code execution, full system compromise
- **Affected Components**: Any system command execution points
- **Likelihood**: Low
- **Severity**: Critical

#### EOP3: Server-Side Template Injection
- **Threat**: Malicious code injected into email templates
- **Impact**: Code execution, information disclosure
- **Affected Components**: Email templates, view rendering
- **Likelihood**: Low
- **Severity**: High

#### EOP4: Dependency Vulnerabilities
- **Threat**: Known vulnerabilities in Rails or gem dependencies
- **Impact**: Various, depending on vulnerability
- **Affected Components**: All components using vulnerable dependencies
- **Likelihood**: Medium
- **Severity**: High

#### EOP5: Mass Assignment Vulnerability
- **Threat**: Attacker adds unexpected parameters to modify protected attributes
- **Impact**: Unauthorized data modification
- **Affected Components**: Controllers (if strong parameters not properly configured)
- **Likelihood**: Low
- **Severity**: Medium

## Countermeasures

### Input Validation and Sanitization

#### CM1: Strong Parameters
- **Addresses**: T1, EOP5
- **Implementation**: Use Rails strong parameters (already implemented)
- **Status**: ✅ Implemented in controllers
- **Code Location**: `ContactsController#secure_params`, `VisitorsController#secure_params`

#### CM2: Email Format Validation
- **Addresses**: T1, S1
- **Implementation**: Regex validation for email format (already implemented)
- **Status**: ✅ Implemented in models
- **Code Location**: `Contact` and `Visitor` models

#### CM3: Content Length Validation
- **Addresses**: DOS2, T1
- **Implementation**: Maximum length constraint on message content
- **Status**: ✅ Implemented (500 character limit)
- **Code Location**: `Contact` model

#### CM4: HTML Sanitization
- **Addresses**: T1 (XSS)
- **Implementation**: Rails automatic HTML escaping in views
- **Status**: ✅ Default Rails behavior
- **Recommendation**: Ensure all user input is properly escaped in views

### Authentication and Authorization

#### CM5: Secure Credential Storage
- **Addresses**: S2, S3, ID2
- **Implementation**: Store secrets in encrypted secrets.yml or environment variables
- **Status**: ⚠️ Partial - using Rails secrets
- **Recommendation**: Migrate to Rails 5.2+ credentials system or external secret management (HashiCorp Vault, AWS Secrets Manager)

#### CM6: API Key Rotation
- **Addresses**: S2
- **Implementation**: Regular rotation of MailChimp API keys
- **Status**: ❌ Not implemented
- **Recommendation**: Establish quarterly API key rotation policy

#### CM7: SMTP Authentication
- **Addresses**: S3
- **Implementation**: Use authenticated SMTP with TLS
- **Status**: ⚠️ Configuration dependent
- **Recommendation**: Enforce TLS for all SMTP connections

### Data Protection

#### CM8: HTTPS Enforcement
- **Addresses**: ID1, ID5, T2
- **Implementation**: Force SSL in production, HSTS headers
- **Status**: ⚠️ Configuration dependent
- **Recommendation**: Add `config.force_ssl = true` in production environment

#### CM9: Security Headers
- **Addresses**: ID5
- **Implementation**: Set CSP, X-Frame-Options, X-Content-Type-Options headers
- **Status**: ❌ Not implemented
- **Recommendation**: Add security headers middleware (rack-attack, secure_headers gem)

#### CM10: Minimize Logging of PII
- **Addresses**: ID1
- **Implementation**: Filter sensitive parameters from logs
- **Status**: ⚠️ Partial - Rails default filtering
- **Recommendation**: Configure `filter_parameters` to exclude email and content

#### CM11: Secure Cookie Configuration
- **Addresses**: T2, ID5
- **Implementation**: HttpOnly, Secure, and SameSite cookie flags
- **Status**: ✅ Rails default configuration
- **Recommendation**: Verify cookie settings in production

### Rate Limiting and DoS Prevention

#### CM12: Rate Limiting
- **Addresses**: DOS1, DOS2, DOS3
- **Implementation**: Request rate limiting per IP address
- **Status**: ❌ Not implemented
- **Recommendation**: Implement rack-attack gem with rate limits (e.g., 10 requests/minute per IP)

#### CM13: CAPTCHA Implementation
- **Addresses**: DOS1, S1
- **Implementation**: Add reCAPTCHA to forms
- **Status**: ❌ Not implemented
- **Recommendation**: Add reCAPTCHA v3 for bot detection

#### CM14: Request Timeout Configuration
- **Addresses**: DOS3
- **Implementation**: Configure Puma timeouts appropriately
- **Status**: ✅ Default configuration
- **Recommendation**: Review and tune timeout settings

#### CM15: External Service Timeout Handling
- **Addresses**: DOS4
- **Implementation**: Set reasonable timeouts for MailChimp and SMTP calls
- **Status**: ⚠️ Partial
- **Recommendation**: Implement circuit breaker pattern for external service calls

### Audit and Monitoring

#### CM16: Security Event Logging
- **Addresses**: R1, R2, R3
- **Implementation**: Log all form submissions with timestamp and IP
- **Status**: ⚠️ Partial - basic Rails logging
- **Recommendation**: Implement structured logging with security context

#### CM17: Error Monitoring
- **Addresses**: ID3, DOS4
- **Implementation**: Centralized error tracking and alerting
- **Status**: ❌ Not implemented
- **Recommendation**: Integrate error monitoring service (Sentry, Rollbar, Honeybadger)

#### CM18: Access Log Retention
- **Addresses**: R3
- **Implementation**: Retain access logs for security analysis
- **Status**: ⚠️ Configuration dependent
- **Recommendation**: Define log retention policy (90 days minimum)

### Dependency Management

#### CM19: Dependency Scanning
- **Addresses**: EOP4
- **Implementation**: Regular scanning for vulnerable dependencies
- **Status**: ❌ Not implemented
- **Recommendation**: Integrate Bundler Audit or Dependabot for automated scanning

#### CM20: Regular Updates
- **Addresses**: EOP4
- **Implementation**: Keep Rails and gems up to date
- **Status**: ⚠️ Using older versions (Rails 5.1.2, Ruby 2.4.1)
- **Recommendation**: Upgrade to supported Ruby and Rails versions (Ruby 3.x, Rails 7.x)

#### CM21: Minimal Dependencies
- **Addresses**: EOP4
- **Implementation**: Only include necessary gems
- **Status**: ✅ Reasonable dependency list
- **Recommendation**: Periodic review of unused dependencies

### Secure Configuration

#### CM22: Separate Development and Production Secrets
- **Addresses**: S2, S3, ID2
- **Implementation**: Different credentials for each environment
- **Status**: ✅ Standard practice
- **Recommendation**: Document secret management process

#### CM23: File Permission Hardening
- **Addresses**: T3, ID2
- **Implementation**: Restrict access to configuration and log files
- **Status**: ⚠️ Deployment dependent
- **Recommendation**: Set restrictive permissions (600 for secrets, 640 for logs)

#### CM24: Disable Directory Listing
- **Addresses**: ID3, ID4
- **Implementation**: Prevent web server directory browsing
- **Status**: ✅ Default Rails/Puma behavior
- **Recommendation**: Verify production web server configuration

### Code Security

#### CM25: Output Encoding
- **Addresses**: T1 (XSS)
- **Implementation**: Use Rails HTML escaping helpers
- **Status**: ✅ Default Rails behavior
- **Recommendation**: Code review to ensure proper usage

#### CM26: SQL Injection Prevention
- **Addresses**: EOP1
- **Implementation**: Use ActiveRecord query interface, avoid raw SQL
- **Status**: ✅ No direct SQL queries in current code
- **Recommendation**: Maintain this practice

#### CM27: Command Injection Prevention
- **Addresses**: EOP2
- **Implementation**: Avoid system calls with user input
- **Status**: ✅ No system calls in current code
- **Recommendation**: Code review for any future command execution

#### CM28: Safe Email Rendering
- **Addresses**: EOP3, T1
- **Implementation**: Sanitize contact form content in emails
- **Status**: ⚠️ Review needed
- **Recommendation**: Review email templates for proper escaping

### Compliance and Privacy

#### CM29: Privacy Policy
- **Addresses**: ID1, R2
- **Implementation**: Clear privacy policy for data collection
- **Status**: ❌ Not documented
- **Recommendation**: Create privacy policy page addressing GDPR requirements

#### CM30: Consent Mechanism
- **Addresses**: R2
- **Implementation**: Explicit consent for newsletter subscription
- **Status**: ✅ User-initiated form submission
- **Recommendation**: Add explicit checkbox for consent with privacy policy link

#### CM31: Data Retention Policy
- **Addresses**: ID1
- **Implementation**: Define retention for logs and any stored data
- **Status**: ❌ Not documented
- **Recommendation**: Document data retention and deletion procedures

#### CM32: Unsubscribe Mechanism
- **Addresses**: R2
- **Implementation**: MailChimp handles unsubscribe links
- **Status**: ✅ MailChimp default functionality
- **Recommendation**: Verify unsubscribe links in email templates

## Summary

The Support Console application has a relatively small attack surface with two primary public-facing features. The main security concerns are:

1. **High Priority**: Implement rate limiting and CAPTCHA to prevent abuse
2. **High Priority**: Upgrade Ruby and Rails to supported versions
3. **High Priority**: Implement comprehensive security headers
4. **Medium Priority**: Enhance audit logging for compliance
5. **Medium Priority**: Set up automated dependency scanning
6. **Medium Priority**: Review and minimize PII logging
7. **Low Priority**: Document privacy and data handling policies

The application benefits from Rails' built-in security features but requires additional hardening for production deployment. Most critical threats can be mitigated through proper configuration, rate limiting, and keeping dependencies current.

## Review and Maintenance

- **Document Version**: 1.0
- **Last Updated**: 2024
- **Review Frequency**: Quarterly
- **Next Review Date**: TBD
- **Document Owner**: Security Team
- **Approvers**: Development Team Lead, Security Team Lead

This threat model should be reviewed and updated when:
- New features are added
- Dependencies are significantly changed
- Security incidents occur
- Regulatory requirements change
- At least quarterly as part of security review process
