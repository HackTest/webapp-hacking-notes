# authentication hacking

### SQL Injection
**Description**  
Hacker uses 'strange' username to bypass password validation and get unauthenticated access to user's account.

**Case Analysis**  
Suppose in webapp backend, the authentication logic is as follows.

Case 1: Correct SQL Syntax | Access to User's account
```
SELECT * FROM users WHERE email='alice@example.com' and password='abc'
```
If we add an extra ' to the SQL query, the backend will throw an execption and the user will be rejected to get access to the account.

Case 2: Incorrect SQL Syntax | No Access to User's account
```
SELECT * FROM users WHERE email='alice@example.com'' and password='abc'
```
However, if the hacker input a tricky username, it will comment out the password validation and the backend will throw no exception.

Case 3: Incorrect SQL Syntax | Access to User's account
```
SELECT * FROM users WHERE email='alice@example.com' -- ' and password='abc'
```
which is equivalent to 
```
SELECT * FROM users WHERE email='alice@example.com'
```

**Defend**  
Sanitization is a good way to defend the SQL  injection. By using the placeholder of variables, if the password is missing, it will throw exceptions, which make SQL injection fail.
```
dbQuery("SELECT * FROM users where email=? and password=?“);
dbBindParameters("ss", $email, $password);
dbStatementExecute();
```

### Dictionary Attack
**Description**  
It generally means automated password guessing.

**Case Analysis**  
1. Hacker uses social engineer methods and know user's account name: alice@example.com  
2. Hacker uses list of common passwords to attempt loggin
3. Hack waits for web app's response to see whether the password is correct or not

**Hydra** is a good tool to automate dictionary attack. A typical script is like this: 
```
hydra example.com -L emails.txt -P passwords.txt https-post-form "/login.php:email=^USER^&password=^PASS^:Invalid password" -S
```

**Defend**  
**CAPTCHA** is commonly used to defend dictionary attack. Since it provide some easy tests (visual recognition) to tell if the user is a real person or computer.


### HTTPS Enforcement
**Description**  
Web Application should always redirect a http request to a https request.

**Case Analysis**  
HTTPS NOT ENFORCED: http://example.com/login.php => http://example.com/login.php  
HTTPS ENFORCED: http://example.com/login.php => https://example.com/login.php  

**Summary**  
HTTP is insecure because the credential is transmitted in plaintext. We should always use HTTPS protocol to protect user's privacy. 

### Session Regeneration
**Description**  
When the user is logged out from the system, the attacker could impersonate the user using the same session ID. 

**Case Analysis**  
1. User is logged out
2. Hacker learned user's session ID using tools like Burp Suite intercept(non-authenticated)
3. User logs in
4. User is logged (authenticated)

**Defend**  
Regenerate cookie with session ID at every time of authentication. Note that only HTTPS is not enough. We also need secure session management.
### User Enumeration
**Description**  
Attacker do enumerations of list of usernames (emails) and could tell if an username is registered based on the web app's responses.

**Case Analysis**  
1. When entered "alice@example.com", the web app returns "invalid email"
2. when entered "bob@example.com" with wrong password, the web app return's "invalid password"
3. The hacker could tell "bob@example.com" is registered.

**Defend**  
The web app should return the same response "invalid email or password" in the two scenarios.

### Best Practices
**Strong password policy**  
Strong password = length + randomness. 

**Two-factor authentication (2FA)**  
2FA = what user knows (password) + what user has (mobile device)
one-time password(verification code) sent to user's mobile device.
