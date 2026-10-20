# Kodhe Email

A refactored version of the CodeIgniter 3 `Email` library under the `Kodhe\Framework\Email` namespace. Sends email via the `mail()` protocol, Sendmail, or SMTP, with CC/BCC support (including batched mass BCC), attachments, MIME headers, text/html mailtype, word-wrapping, and a built-in debugger. The transport layer is modularized (`MailTransport`, `SendmailTransport`, `SmtpTransport`), making it easy to extend or mock during testing.

## Installation

```bash
composer require kodhe/email
```

Requirements: PHP >= 8.1. No mandatory external dependencies (SMTP uses PHP's built-in sockets).

## Quick Start

```php
<?php

declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Kodhe\Framework\Email\Email;

$email = new Email();

$email->from('admin@example.com', 'Admin')
      ->to('user@example.com')
      ->subject('Hello')
      ->message('This is the email body.')
      ->send();
```

The fluent API returns `EmailInterface`, so calls can be chained as above. `send()` returns a `bool`.

## Configuration

Via the constructor or `initialize()`:

```php
$email = new Email([
    'protocol'  => 'smtp',            // mail | sendmail | smtp
    'smtp_host' => 'smtp.example.com',
    'smtp_port' => 587,
    'smtp_user' => 'user@example.com',
    'smtp_pass' => 'password',
    'SMTPAuth'  => true,
    'mailtype'  => 'html',            // text | html
    'charset'   => 'UTF-8',
    'wordwrap'  => true,
    'wrapchars' => 76,
    'priority'  => 3,                 // 1 (highest) to 6
]);
```

## Main Methods

| Method | Description |
|---|---|
| `from(string $from, string $name = '')` | Sender address |
| `replyTo(string $replyto, string $name = '')` | Reply-to address |
| `to($to)` / `cc($cc)` | Single recipient or array |
| `bcc($bcc, int $limit = 0)` | BCC; `$limit > 0` = mass BCC in batches |
| `subject(string $subject)` / `message(string $body)` | Subject and body |
| `attach(string $filename, string $disposition = '', ?string $newname = null, string $mime = '')` | Attachment (can be called repeatedly) |
| `send(bool $autoClear = true): bool` | Send; auto-clears after delivery |
| `clear(bool $clearAttachments = false)` | Resets the message state |
| `printDebugger(array $include = ['headers', 'subject', 'body']): string` | Debug output without actually sending |

### HTML + attachment example

```php
$email = new Email(['protocol' => 'mail', 'mailtype' => 'html']);

$email->from('admin@example.com')
      ->to(['a@example.com', 'b@example.com'])
      ->subject('Weekly Report')
      ->message('<h1>Report</h1><p>PDF attached.</p>')
      ->attach('/path/to/report.pdf')
      ->send();
```

### Suppressing real sends during development

```php
// Don't call send(); just inspect the header/body output:
echo $email->printDebugger();
```

## Struktur Direktori

```
src/
├── Email.php                     # Main class (CI3-compatible API)
├── Contracts/                    # EmailInterface, TransportInterface
├── Transports/                   # MailTransport, SendmailTransport, SmtpTransport
├── Message/                      # EmailMessage, Attachment, HeaderCollection
├── Traits/                       # ConfigurableTrait, DebugTrait
├── language/email_lang.php
└── helpers/email.php             # valid_email(), send_email()
```

## Migration Notes from the Old Namespace

Older documentation/tests used `Kodhe\Library\Email\Email` along with legacy CI3-style methods (`reply_to()`, `set_mailtype()`, the public `word_wrap` property). In modern Composer versions:

```php
use Kodhe\Framework\Email\Email;
```

Configuration is done via an array (see the Configuration section), and method names use camelCase (`replyTo()`). The PSR-4 mapping `Kodhe\Framework\Email\` → `email/src/` is declared in `email/composer.json`.
