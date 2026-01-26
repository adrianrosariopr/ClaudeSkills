<overview>
Mailgun supports email templates with dynamic content using Handlebars syntax. Templates can be stored in Mailgun or managed locally.
</overview>

<creating_templates>
## Creating Templates

<via_dashboard>
**Via Mailgun Dashboard:**
1. Go to **Sending → Templates**
2. Click **Create Template**
3. Enter template name (used in API)
4. Design with visual editor or HTML
5. Add variables using `{{variable_name}}`
6. Save template
</via_dashboard>

<via_api>
**Via API (Node.js):**
```javascript
await mg.templates.create(domain, {
  name: 'welcome-email',
  description: 'Welcome new users',
  template: `
    <html>
      <body>
        <h1>Welcome, {{name}}!</h1>
        <p>Thanks for joining {{company}}.</p>
      </body>
    </html>
  `
});
```
</via_api>
</creating_templates>

<variable_syntax>
## Variable Syntax

<basic_variables>
**Basic variables:**
```html
<p>Hello {{firstName}},</p>
<p>Your order #{{orderNumber}} is confirmed.</p>
```
</basic_variables>

<nested_variables>
**Nested objects:**
```html
<p>Ship to: {{address.street}}, {{address.city}}</p>
```
</nested_variables>

<default_values>
**Default values:**
```html
<p>Hello {{firstName default="Customer"}},</p>
```
</default_values>
</variable_syntax>

<handlebars_helpers>
## Handlebars Helpers

<if_helper>
**Conditional content (if):**
```html
{{#if premium}}
  <p>Thanks for being a premium member!</p>
{{else}}
  <p>Upgrade to premium for more features.</p>
{{/if}}
```
</if_helper>

<equal_helper>
**Equality check (equal):**
```html
{{#equal status "shipped"}}
  <p>Your order is on its way!</p>
{{/equal}}

{{#equal plan "enterprise"}}
  <p>Enterprise support: support@company.com</p>
{{/equal}}
```

Note: `equal` compares as strings, so `5` equals `"5"`.
</equal_helper>

<unless_helper>
**Inverse conditional (unless):**
```html
{{#unless verified}}
  <p>Please verify your email address.</p>
{{/unless}}
```
</unless_helper>

<each_helper>
**Loop through arrays (each):**
```html
<h2>Your Order</h2>
<ul>
{{#each items}}
  <li>{{this.name}} - ${{this.price}}</li>
{{/each}}
</ul>
<p>Total: ${{total}}</p>
```
</each_helper>

<with_helper>
**Change context (with):**
```html
{{#with user}}
  <p>Name: {{firstName}} {{lastName}}</p>
  <p>Email: {{email}}</p>
{{/with}}
```
</with_helper>
</handlebars_helpers>

<sending_with_template>
## Sending with Templates

<nodejs>
**Node.js:**
```javascript
await mg.messages.create(domain, {
  from: 'noreply@yourdomain.com',
  to: ['user@example.com'],
  subject: 'Welcome to {{company}}',  // Subject can also use variables
  template: 'welcome-email',
  'h:X-Mailgun-Variables': JSON.stringify({
    firstName: 'John',
    company: 'Acme Corp',
    premium: true,
    items: [
      { name: 'Widget', price: 9.99 },
      { name: 'Gadget', price: 19.99 }
    ],
    total: 29.98
  })
});
```
</nodejs>

<alternative_method>
**Alternative: t:variables parameter:**
```javascript
await mg.messages.create(domain, {
  from: 'noreply@yourdomain.com',
  to: ['user@example.com'],
  subject: 'Your Order',
  template: 'order-confirmation',
  't:variables': JSON.stringify({
    name: 'John',
    orderNumber: '12345'
  })
});
```
</alternative_method>
</sending_with_template>

<recipient_variables>
## Recipient Variables (Batch Personalization)

Send one API call to multiple recipients with individual personalization:

```javascript
await mg.messages.create(domain, {
  from: 'news@yourdomain.com',
  to: ['alice@example.com', 'bob@example.com', 'carol@example.com'],
  subject: 'Hello %recipient.name%!',
  text: 'Hi %recipient.name%, your code is %recipient.code%.',
  html: '<p>Hi %recipient.name%, your code is <strong>%recipient.code%</strong>.</p>',
  'recipient-variables': JSON.stringify({
    'alice@example.com': {
      name: 'Alice',
      code: 'ALICE123'
    },
    'bob@example.com': {
      name: 'Bob',
      code: 'BOB456'
    },
    'carol@example.com': {
      name: 'Carol',
      code: 'CAROL789'
    }
  })
});
```

<syntax_note>
**Note:** Recipient variables use `%recipient.variable%` syntax, not Handlebars `{{}}`.

This is different from template variables:
- Template variables: `{{name}}` - for Mailgun-stored templates
- Recipient variables: `%recipient.name%` - for inline content personalization
</syntax_note>
</recipient_variables>

<template_versions>
## Template Versions

Mailgun supports template versions for A/B testing or rollbacks:

```javascript
// Create version
await mg.templates.createVersion(domain, 'welcome-email', {
  tag: 'v2',
  template: '<html>...<html>',
  active: false
});

// Activate version
await mg.templates.updateVersion(domain, 'welcome-email', 'v2', {
  active: true
});

// Send specific version
await mg.messages.create(domain, {
  to: ['user@example.com'],
  template: 'welcome-email',
  't:version': 'v2'
});
```
</template_versions>

<best_practices>
## Best Practices

<design>
**Design:**
- Use inline CSS (not external stylesheets)
- Provide both HTML and text versions
- Test across email clients (Litmus, Email on Acid)
- Keep width under 600px for mobile
</design>

<personalization>
**Personalization:**
- Always provide default values for optional variables
- Validate data before sending
- Don't expose sensitive data in emails
</personalization>

<testing>
**Testing:**
- Preview templates in Mailgun dashboard
- Send test emails before production
- Test with various data scenarios
</testing>
</best_practices>
