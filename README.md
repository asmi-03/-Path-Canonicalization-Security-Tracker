# -Path-Canonicalization-Security-Tracker
Path Canonicalization & Security Tracker
This Node.js Express server tracks incoming URL requests, canonicalizes their paths using POSIX rules, detects URL path redundancy, and implements a secure file-serving mechanism.

🚀 Key Goals
URL Canonicalization: Resolve redundant URL forms (e.g., /a//b, /a/./b, /a/b/../c) to a single canonical path (/a/c).

Redundancy Detection: Identify when multiple original URLs map to the same canonical path.

Security: Prevent Directory Traversal attacks when serving files.

Correct path Usage: Demonstrate when to use path.posix vs. the default path module.

1️⃣ Project Setup
Initialize Project (if not already done):

Bash

mkdir path-tracker-app
cd path-tracker-app
# Create package.json with dependencies: express, morgan
npm init -y
npm install express morgan
Create Files/Folders:

path-tracker-app/
├── code11.js          # The main server file
├── public/            # Directory for static files
└── package.json
Setup Static Files (for testing the /static-file/* route):

Bash

mkdir public
# Create a dummy file for testing access
echo "This is a static file." > public/test.txt
Copy Code: Place the provided code into code11.js.

2️⃣ Running the Application
Start the server using Node.js:

Bash

node code11.js
The server will start on http://localhost:4000.

3️⃣ Testing & Observing Redundancy
Visit the following URLs in your browser (or using curl):

Action	URL Visited	Canonical Path (Expected)	Redundancy Status
Visit	http://localhost:4000/about	/about	No
Test Redundancy	http://localhost:4000/about/	/about	Yes (Alias: /about/)
Test Normalization	http://localhost:4000/a//b/./c	/a/b/c	No (on first hit)
Test Traversal	http://localhost:4000/a/b/../c	/a/c	No (on first hit)

Export to Sheets

After visiting the above URLs, navigate to the tracker page:

Check Tracker Page: http://localhost:4000/patterns

On this page, you will see entries like:

/about: Count: 2+, Redundancy? Yes, with aliases like /about and /about/.

/a/b/c: Count: 1+, Redundancy? Yes, with aliases like /a//b/./c.

4️⃣ The Core Logic: The path Module
The code strategically uses two different variants of the Node.js path module:

Context	Module Used	Function	Why this choice?
URL Canonicalization	path.posix	path.posix.normalize(rawPath)	URLs are OS-independent and use forward slashes (/). path.posix ensures consistent, cross-platform normalization (collapsing //, resolving .., etc.) using Unix/POSIX rules.
Filesystem Access	path (default)	path.resolve(publicDir, safeRel)	The file system needs the native OS separator (\ on Windows, / on Linux/macOS). path.resolve() safely constructs an absolute path for file access and is critical for the security check.

Export to Sheets

The /which-path route provides a programmatic explanation of this choice.

5️⃣ Security Implementation (Directory Traversal)
The /static-file/* route implements a vital security measure:

It determines the trusted root directory: const publicDir = path.resolve(__dirname, 'public');

It calculates the final requested file path: const requestedFsPath = path.resolve(publicDir, safeRel);

It blocks traversal:

JavaScript

if (!requestedFsPath.startsWith(publicDir)) {
    return res.status(403).send('Forbidden');
}
This ensures that even if a request contains dangerous path segments like ../../etc/passwd, path.resolve() makes them absolute, and the check prevents the server from accessing any file outside the defined publicDir.






