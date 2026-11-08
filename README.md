# Online Pizza Delivery System

A multi-page ASP.NET Web Forms website for a demo restaurant called "Pizza Delicious". It was built for the *Internet Programming Laboratory* course (CSE 3100) at Khulna University of Engineering & Technology (KUET).

> [!NOTE]
> **Archived coursework, not production software.** Browsing, form submission and account storage are implemented. Ordering was never finished, and the code has serious security flaws, listed under [Security review](#security-review). This repository is a record of undergraduate web-development work and is not part of the owner's research in robust and trustworthy machine learning.
>
> The project has no connection to Pizza Hut. The repository name is informal.

## Implementation status

The [project description](pizza.docx) sets out an ordering platform with admin, manager and customer roles. This table compares that description with the code in this repository.

| Capability | Status | Detail |
| --- | --- | --- |
| Home, menu, services, about and contact pages | Implemented | Content is written directly in the markup. Menu items and prices (in BDT) are hard-coded. |
| Contact and message forms | Implemented | Submissions are saved to the database. |
| Customer registration | Implemented, with a defect | Accounts are saved to a different database from the one login reads. See [Data](#data). |
| Login | Partial | Credentials are checked and the user is redirected, but no session is created. |
| Remembered credentials | Partial | Email and password are kept in cookies for one minute and used to pre-fill the login form. |
| Logged-in navigation, welcome page and logout | Unreachable | This code depends on `Session["Email"]`, which is read on every page but never set. |
| Shopping cart and checkout | Mock-up only | "Add to Cart" links lead to `#` or a static `cart.html` with sample rows. The checkout page does not exist. |
| Admin and manager roles, food upload and approval | Not implemented | |
| Food reviews and ratings | Not implemented | |
| Order confirmation emails and receipts | Not implemented | |
| Online payment | Not implemented | The footer shows bKash, Rocket and Visa/Mastercard logos only. |

## Technology

| Layer | Technology |
| --- | --- |
| Server | ASP.NET Web Forms, C#, .NET Framework 4.6.1 |
| Data access | ADO.NET (`SqlConnection`, `SqlCommand`) |
| Database | SQL Server Express and SQL Server LocalDB |
| Front end | Bootstrap 4.1, Materialize, jQuery 3.2.1, Owl Carousel, AOS, Google Maps JavaScript API |
| Tooling | Visual Studio 2017 solution (Web Site project), Sass with Prepros |

The server-side code is about 390 lines of C# across nine code-behind files. Most of the 432 tracked files are bundled libraries, compiler binaries and images.

## How the site works

### Pages

| Page | Purpose | Server-side behaviour |
| --- | --- | --- |
| [`index.aspx`](pizza/pizza/index.aspx) | Landing page, featured items and a message form | Inserts messages into `Table3` |
| [`menu.aspx`](pizza/pizza/menu.aspx) | Pizza, drinks, burgers and pasta | None beyond the shared navigation check |
| [`services.aspx`](pizza/pizza/services.aspx) | Service highlights | None beyond the shared navigation check |
| [`about.aspx`](pizza/pizza/about.aspx) | Restaurant information and a message form | Inserts messages into `Table3` |
| [`contact.aspx`](pizza/pizza/contact.aspx) | Contact form and map | Inserts messages into `Table2` |
| [`register.aspx`](pizza/pizza/register.aspx) | Sign-up page linked from login | Validates input, then inserts into `Table` |
| [`login.aspx`](pizza/pizza/login.aspx) | Sign-in | Queries `log_in`, sets cookies and redirects to the home page |
| [`Signup.aspx`](pizza/pizza/Signup.aspx) | Second sign-up page, not linked from any other page | Checks for a duplicate email, then inserts into `log_in` |
| [`success.aspx`](pizza/pizza/success.aspx) | Welcome message and logout | Requires `Session["Email"]`, so its logged-in view never appears |

Registration uses ASP.NET validators for matching passwords, email format, 11-digit phone numbers and passwords of 8 to 10 letters and digits containing at least one of each.

### Data

The code uses two databases. Neither the database files nor a schema script are in this repository.

| Database | Connection | Tables | Used by |
| --- | --- | --- | --- |
| `ss` | `con` connection string in [`Web.config`](pizza/pizza/Web.config) (SQL Server Express) | `log_in` | `login.aspx`, `Signup.aspx` |
| `register.mdf` | LocalDB connection hard-coded in each code-behind file | `Table` (accounts), `Table2` (contact messages), `Table3` (site messages) | `register.aspx`, `contact.aspx`, `index.aspx`, `about.aspx` |

Because of this split, an account created on the linked `register.aspx` page cannot be used to log in. Login only reads `log_in`, which is filled by the unlinked `Signup.aspx`.

## Security review

These problems were found by reading the code. They are documented here and have not been fixed, so this code should not be deployed.

| Severity | Issue | Location | Proper fix |
| --- | --- | --- | --- |
| Critical | **SQL injection.** Six of the seven SQL statements concatenate user input. This includes the login query, so authentication can be bypassed. | [login](pizza/pizza/login.aspx.cs#L41), [Signup](pizza/pizza/Signup.aspx.cs#L50), [register](pizza/pizza/register.aspx.cs#L19), [contact](pizza/pizza/contact.aspx.cs#L27), [about](pizza/pizza/about.aspx.cs#L27), [index](pizza/pizza/index.aspx.cs#L27) | Use parameterised queries everywhere, as the `INSERT` in [`Signup.aspx.cs`](pizza/pizza/Signup.aspx.cs#L79) already does. |
| Critical | **Plaintext passwords.** Both account tables store passwords exactly as typed. | [register](pizza/pizza/register.aspx.cs#L19), [Signup](pizza/pizza/Signup.aspx.cs#L81) | Store salted hashes using ASP.NET Identity or PBKDF2. |
| High | **Password stored in a cookie.** The password is written to a `UPWD` cookie in clear text, without `HttpOnly` or `Secure`, and then placed in the login form's `value` attribute. | [write](pizza/pizza/login.aspx.cs#L47-L51), [read](pizza/pizza/login.aspx.cs#L23) | Remove the cookie. Use a server-side session or a signed authentication ticket. |
| Medium | **Validation result ignored.** `register.aspx` declares validators, but the handler never checks `Page.IsValid`, so a crafted request skips them. | [register](pizza/pizza/register.aspx.cs#L17) | Check `Page.IsValid` before writing. |
| Medium | **Unsafe error handling.** Raw exception text is written into an inline script on sign-up, and login discards every exception silently. | [Signup](pizza/pizza/Signup.aspx.cs#L65), [login](pizza/pizza/login.aspx.cs#L74) | Log errors on the server and show a generic message. |
| Low | **Configuration in source.** A Google Maps API key appears in page markup, and `Web.config` has a machine-specific server name and `debug="true"`. | [`Web.config`](pizza/pizza/Web.config) | Move settings to per-environment configuration and restrict the key. |

## Running locally

These steps come from the solution and configuration files and have not been tested on a clean machine. The table definitions are reconstructed from the SQL in the code. Column types and sizes are assumptions.

**Requirements:** Windows, Visual Studio 2017 or later with the ASP.NET workload and the .NET Framework 4.6.1 targeting pack, SQL Server Express, and SQL Server LocalDB.

1. Clone the repository and open [`pizza/pizza.sln`](pizza/pizza.sln).
2. Create a database named `ss`, add the `log_in` table below, and change the `con` connection string in `Web.config` to point at your server.
3. Create `pizza/pizza/App_Data/register.mdf` and add the other three tables below.
4. Press F5 and open `index.aspx`.

```sql
-- In database "ss"
CREATE TABLE log_in (
    Name     NVARCHAR(100),
    Password NVARCHAR(100),
    Email    NVARCHAR(256),
    Phone    NVARCHAR(20)
);

-- In App_Data\register.mdf
CREATE TABLE [Table] (
    Name     NVARCHAR(100),
    Email    NVARCHAR(256),
    Password NVARCHAR(100),
    Phone    NVARCHAR(20)
);

CREATE TABLE [Table2] (
    Name    NVARCHAR(100),
    Email   NVARCHAR(256),
    Subject NVARCHAR(200),
    Message NVARCHAR(MAX)
);

CREATE TABLE [Table3] (
    Name    NVARCHAR(100),
    Email   NVARCHAR(256),
    Message NVARCHAR(MAX)
);
```

To test login, create the account by opening `Signup.aspx` directly, because accounts from `register.aspx` go to the other database.

## Repository layout

```text
.
├── cover.docx               Course cover sheet
├── pizza.docx               Project description
└── pizza/
    ├── pizza.sln            Visual Studio solution
    ├── packages/            NuGet packages
    └── pizza/
        ├── *.aspx           Web Forms pages
        ├── *.aspx.cs        C# code-behind
        ├── *.html           Static versions of some pages, a few still linked
        ├── Web.config       Connection string and compiler settings
        ├── Bin/             Roslyn compiler binaries
        ├── css/ scss/ js/   Template styles and bundled libraries
        └── fonts/ images/   Icon fonts, photos and media
```

## Credits and license

- **Developer:** Md. Shahinur Hasan.
- **Course:** CSE 3100, Internet Programming Laboratory, 3rd year 1st semester, KUET.
- **Original submission:** the [cover sheet](cover.docx) names Chowdhury Shezan Reza, Department of ECE, as the submitting student, with a submission date of 11 March 2020.
- **This repository:** uploaded in a single commit by Md. Shahinur Hasan on 5 February 2024.
- **Front-end design:** based on third-party HTML templates. The main pages use a Bootstrap 4 template with `ftco-` class names, and the login and sign-up pages use a Tooplate stylesheet. The original template credits are not present in this copy.

No license file is included, so default copyright applies. Bundled libraries, fonts and templates remain under their own licenses.
