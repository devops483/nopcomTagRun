# Setting Up nopCommerce on IIS with Multi-Store, SSL, and Domain Configuration

## Introduction

In this blog post, we will walk through setting up nopCommerce from the official GitHub repository and hosting it on IIS. We will also cover how to configure multiple stores, enable SSL using self-signed certificates, and access each store with its own domain, language, and currency.

This guide is perfect for developers or businesses looking to simulate a multi-tenant nopCommerce setup in a local environment before deploying to production.

---

## 1. Cloning nopCommerce from GitHub

1. Visit the official GitHub repo: [https://github.com/nopSolutions/nopCommerce](https://github.com/nopSolutions/nopCommerce)
2. Clone the repo:

   ```bash
   git clone https://github.com/nopSolutions/nopCommerce.git
   ```
3. Open the solution in Visual Studio.
4. Build the solution.

---

## 2. Setting up nopCommerce on IIS

1. Open IIS Manager.
2. Add a new site:

   * Site name: `nopcom`
   * Physical path: `<path-to-nopCommerce>/Presentation/Nop.Web`
   * Port: 80
   * Hostname: `nopcom.local`
3. Create a new Application Pool targeting .NET CLR v6 or latest.
4. Assign the app pool to the site.

---

## 3. Editing Hosts File

Edit `C:\Windows\System32\drivers\etc\hosts` and add:

```
127.0.0.1 nopcom.local
```

---

## 4. Creating and Binding a Self-Signed SSL Certificate

1. Use PowerShell to create a cert:

   ```powershell
   New-SelfSignedCertificate -DnsName "nopcom.local" -CertStoreLocation "cert:\LocalMachine\My"
   ```
2. In IIS:

   * Bind a new HTTPS binding to `nopcom.local`
   * Assign the self-signed certificate
   * Enable SNI

---

## 5. Running nopCommerce Installation Wizard

1. Navigate to `https://nopcom.local`
2. Fill out DB and admin details.
3. Complete the installation.

---

## 6. Creating Multiple Stores

1. In Admin → Configuration → Stores
2. Add three stores:

   * `nopcom1.local`
   * `nopcom2.local`
   * `nopcom3.local`
3. Each store should have a unique URL.

Update hosts file:

```
127.0.0.1 nopcom1.local
127.0.0.1 nopcom2.local
127.0.0.1 nopcom3.local
```

---

## 7. Configuring Multi-Language and Multi-Currency

1. Admin → Configuration → Language → Add three languages (e.g., English, French, Hindi)
2. Admin → Configuration → Currency → Add USD, EUR, INR
3. Optionally assign specific languages and currencies per store.

---

## 8. Enabling SSL in nopCommerce

1. Admin → Configuration → Stores
2. Edit each store and enable the SSL checkbox

---

## 9. Creating More Self-Signed SSL Certificates

Repeat for each domain:

```powershell
New-SelfSignedCertificate -DnsName "nopcom1.local" -CertStoreLocation "cert:\LocalMachine\My"
New-SelfSignedCertificate -DnsName "nopcom2.local" -CertStoreLocation "cert:\LocalMachine\My"
New-SelfSignedCertificate -DnsName "nopcom3.local" -CertStoreLocation "cert:\LocalMachine\My"
```

Assign each certificate in IIS bindings using SNI.

---

## 10. Testing the Setup

* Visit:

  * `https://nopcom1.local`
  * `https://nopcom2.local`
  * `https://nopcom3.local`
* Ensure:

  * HTTPS is active
  * SSL is enabled in admin panel
  * Correct store loads
  * Language and currency options appear correctly

---

## 11. Troubleshooting & Errors Faced

*To be filled once you provide the actual error messages.*

---

## Conclusion

With this setup, you’ve successfully configured nopCommerce with multiple stores on a local IIS server, each with its own domain, self-signed SSL certificate, languages, and currencies. This mirrors a real-world enterprise eCommerce solution in a local test environment.

In a production environment, replace self-signed certificates with valid ones and configure DNS records accordingly.
