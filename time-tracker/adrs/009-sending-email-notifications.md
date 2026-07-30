# 009: Sending email notifications
 
## Status
Proposed

## Context
We need to set up automatic email notifications to corporate mail (VK WorkMail) for employees. These notifications should be sent when someone creates, updates, or deletes any of these events: away with make-up time, sick leave, vacation, or other events.

SMTP Limitations (VK WorkMail):
- SMTP access is only available in paid plans.
- Limit: max 30 recipients per email (this includes all addresses in "To", "Carbon Copy", and "Blind Carbon Copy" fields).
- Limit: about 500 emails per hour (contact support for the exact limit).
- Risk: automated bulk emails may be marked as spam.

[Source about 30 recipients](https://help.mail.ru/mail/letters/sent/send/?platform=desktop#receiver)

## Decision (Send one email to all recipients (employees))
Sends a single message with all employees listed as recipients. 

### How to get around the 30-recipient limit?
You can split the message and send it multiple times with different recipient groups. For example, if the company has 50 employees, you would need to send two messages: the first one with 30 people, and the second one with the remaining 20.

### Advantages:
- Avoids the 30-recipient limit per email
- Low load on API and SMTP
- Hourly email limit is used up slowly

### Alternatives

## Shared Email with Delegated Access
We create a dedicated mailbox (e.g., notifications@tourmalinecore.com). The time-tracker sends all notifications to this single mailbox. Employees are then given read access to it via the "Assistant" feature in VK WorkMail.

### Advantages:
- Avoids the 30-recipient limit per email
- Hourly email limit is used up slowly
- Low risk of being marked as spam
- Low load on API and SMTP

### Disadvantages:
- Requires creating an additional email (must be paid every month)

### Things to check:
- Will each employee have their own read/unread status for emails? Or if one person reads an email, will it be marked as read for everyone else too?
- How will this actually appear in each employee's email? Will there be a separate folder, or will it work differently?

### Sending individually to each employee
Sends a separate email to each employee's personal email.

### Advantages:
- Avoids the 30-recipient limit per email

### Disadvantages:
- Higher load on API and SMTP (more calls made)
- Hourly email limit is used up quickly
- Higher risk of being marked as spam (many emails sent at once)
- Risk that some emails won't reach the recipients

## Use the "Mailing" tool in VK WorkMail
Mailing are an additional feature of the VK WorkSpace platform for mass corporate email campaigns. This tool is well-suited for marketing purposes (e.g., mass mailing of news to clients, promotions, etc.), but not for automatic internal company notifications.

### Advantages:
- Minimal risk of being marked as spam

### Disadvantages:
- Can only be started manually through the UI interface
- No option for automatic sending via API/SMTP
- Paid feature