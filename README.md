# NextAuth Example

This project demonstrates how to set up Email Authentication and Google OAuth in a Next.js 14 application. It includes a setup for HTTPS in development using `mkcert` to ensure secure connections.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [HTTPS Setup with mkcert](#https-setup-with-mkcert)
- [Custom HTTPS Server](#custom-https-server)
- [Project Structure](#project-structure)
- [Scripts](#scripts)
- [Notes](#notes)

## Prerequisites

Ensure you have the following installed:
- Node.js 14 or newer
- NPM or Yarn
- `mkcert` installed globally on your system
- Google API credentials (Client ID and Client Secret) for OAuth

### Install mkcert

If not installed yet, run:

```bash
# Install mkcert on Ubuntu
sudo apt install libnss3-tools
sudo apt install mkcert
```

## Getting Started

1. **Clone the Repository**
   ```bash
   git clone <repository-url> next-auth
   cd next-auth
   ```

2. **Install Dependencies**
   ```bash
   npm install
   ```

3. **Set up Environment Variables**  
   Create a `.env.local` file in the root directory and add your environment variables:
   ```
   GOOGLE_CLIENT_ID=your_google_client_id
   GOOGLE_CLIENT_SECRET=your_google_client_secret
   NEXTAUTH_URL=https://localhost:3000
   ```

4. **Generate HTTPS Certificates with mkcert**  
   Run the following command to generate local certificates:
   ```bash
   mkcert localhost
   ```
   This will create `localhost.pem` and `localhost-key.pem` files in your project directory.

5. **Run the Development Server**  
   Use the command:
   ```bash
   npm run dev
   ```
   This will start the development server on `https://localhost:3000` with HTTPS enabled.

## HTTPS Setup with mkcert

To enable HTTPS in development:
1. Install `mkcert` globally, as described above
2. Run `mkcert -install` to set up a local certificate authority
3. Use `mkcert localhost` to generate the certificate and key files
4. These files are used in the custom HTTPS server script (`server.js`)

## Custom HTTPS Server

The custom server is created using the built-in `https` module in Node.js. It loads the generated certificates and serves the Next.js app over HTTPS.

**server.js**
```javascript
import { createServer } from 'https';
import { parse } from 'url';
import next from 'next';
import fs from 'fs';
import path from 'path';
import { fileURLToPath } from 'url';

// __dirname workaround in ES modules
const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

const httpsOptions = {
  key: fs.readFileSync(path.join(__dirname, 'localhost-key.pem')),
  cert: fs.readFileSync(path.join(__dirname, 'localhost.pem')),
};

const dev = process.env.NODE_ENV !== 'production';
const app = next({ dev });
const handle = app.getRequestHandler();

app.prepare().then(() => {
  createServer(httpsOptions, (req, res) => {
    const parsedUrl = parse(req.url, true);
    handle(req, res, parsedUrl);
  }).listen(3000, (err) => {
    if (err) throw err;
    console.log('> Server running at https://localhost:3000');
  });
});
```

## Project Structure

```
next-auth/
├── pages/
│   ├── api/
│   │   └── auth/     # NextAuth API Route
│   └── index.js      # Homepage
├── server.js         # Custom HTTPS server
├── .env.local        # Environment variables
├── localhost.pem     # SSL Certificate
├── localhost-key.pem # SSL Certificate Key
├── package.json      # Project scripts and dependencies
└── README.md         # Documentation
```

## Scripts

- `npm run dev`: Starts the development server with HTTPS enabled
- `npm run build`: Builds the application for production
- `npm run start`: Starts the application in production mode
- `npm run lint`: Runs the linter to check for code quality

## Notes

- **Email Authentication and Google OAuth:** Set up using NextAuth.js in the `pages/api/auth/[...nextauth].js` route. Ensure the environment variables for Google OAuth are correctly set in `.env.local`.
- **HTTPS in Development:** Using HTTPS in development helps simulate real-world scenarios and allows testing of secure cookie behaviors, OAuth redirects, and more.