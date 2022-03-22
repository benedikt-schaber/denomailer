## Deno SMTP mail client

### IMPORTANT SECURITY INFORMATION

PLEASE update to a version >= 0.8! 0.8 has a problem where malformed mails could
potatialy allow attackers to create a mail (with linebreaks) to send unwanted
SMTP commands. This could result in authentic phishing attacks! Whith no way for
the user to identify that this is a phishing mail! Or that this mail contains a
dangorus attachment!

Also make shure that Mails are sent one after the other as they can corrupt each
others data!

### Allowed Mail Formats

A single Mail `mail@example.de` with a name `NAME` can be encoded in the
following ways:

1. `"name@example.de"`
2. `"<name@example.de>"`
3. `"NAME <name@example.de>"`
4. `{mail: "name@example.de"}`
5. `{mail: "name@example.de", name: "NAME"}`

Where 1-3 is called a "MailString".

Multiple Mails can be an Array of the above OR a object that maps names to mails
for example:

`{"P1": "p1@example.de", "P2": "p2@example.de"}` we call this a MailObject.

For the fields

1. `from`, `replyTo` we only allow a "MailString".
2. `to`, `cc`, `bcc` we allow a MailObject a Array of single Mails or a single
   Mail.

### Sending multiple mails

Note that for race-condition reasons we can't send multiple mails at once.
Because of that if send is allready called and still processing a mail
`client.send` will que that sending.

### Example

```ts
import { SmtpClient, quotedPrintableEncode } from "https://deno.land/x/denomailer/mod.ts";

const client = new SmtpClient();

await client.connect({
  hostname: "smtp.163.com",
  port: 25,
  username: "username",
  password: "password",
});

await client.send({
  from: "mailaddress@163.com",
  to: "Me <to-address@xx.com>",
  cc: [
    "name@example.de",
    "<name@example.de>",
    "NAME <name@example.de>",
    {mail: "name@example.de"},
    {mail: "name@example.de", name: "NAME"}
  ],
  bcc: {
    "Me": "to-address@xx.com"
  },
  subject: "Mail Title",
  content: "Mail Content",
  html: "<a href='https://github.com'>Github</a>",
  date: "12 Mar 2022 10:38:05 GMT",
  priority: "high",
  replyTo: 'mailaddress@163.com',
  attachments: [
    { encoding: "text"; content: 'Hi', contentType: 'text/plain', filename: 'text.txt' },
    { encoding: "base64"; content: '45dasjZ==', contentType: 'image/png', filename: 'img.png' },
    {
      content: new Uint8Array([0,244,123]),
      encoding: "binary",
      contentType: 'image/jpeg', 
      filename: 'bin.png'
    }
  ],
  mimeContent: [
    {
      mimeType: 'application/markdown',
      content: quotedPrintableEncode('# Title\n\nHello World!'),
      transferEncoding: 'quoted-printable'
    }
  ]
  
});

await client.close();
```

#### TLS connection

```ts
await client.connectTLS({
  hostname: "smtp.163.com",
  port: 465,
  username: "username",
  password: "password",
});
```

#### Use in Gmail

```ts
await client.connectTLS({
  hostname: "smtp.gmail.com",
  port: 465,
  username: "your username",
  password: "your password",
});

await client.send({
  from: "someone@163.com", // Your Email address
  to: "someone@xx.com", // Email address of the destination
  subject: "Mail Title",
  content: "Mail Content，maybe HTML",
});

await client.close();
```

### Configuring your client

You can pass options to your client through the `SmtpClient` constructor.

```ts
import { SmtpClient } from "https://deno.land/x/denomailer/mod.ts";

//Defaults
const client = new SmtpClient({
  console_debug: true, // enable debugging this is good while developing should be false in production as Authentication IS LOGGED TO CONSOLE!
  unsecure: true, // allow unsecure connection to send authentication IN PLAIN TEXT and also mail content!
});
```

## TLS issues
When getting TLS errors make shure:
1. you use the correct port (mostly 25, 587, 465)
2. the server supports STARTTLS when using `client.connect`
3. the server supports TLS when using `client.connectTLS`
4. Use the command `openssl s_client -debug -starttls smtp -crlf -connect your-host.de:587` or `openssl s_client -debug -crlf -connect your-host.de:587` and get the used cipher this should be a cipher with "forward secrecy". Check the status of the cipher on https://ciphersuite.info/cs/ . If the cipher is not STRONG this is an issue with your mail provider so you have to contact them to fix it.
5. Feel free to create issues if you are ok with that share the port and host so a proper debug can be done.
6. We can only support TLS where Deno supports it and Deno uses rustls wich explicitly not implemented some "weak" ciphers.
