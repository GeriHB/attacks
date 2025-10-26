# Password reset poisoning

We can manipulate a vulnerable website into generating a password reset link pointing to a domain under our control.

Using this we an steal the secret tokens required to reset arbitrary users passwords.

**How does this process work?**

When a user wants to reset the password, the website generates a unique, high-entropy token, whih is associates with the user's account on the back-end.

Then an email is sent with this token as a query parameter in the URL. After the usage this token is destroyed.

But its seurity relies on the principle that only that user has access to the email inbox, and to this unique token. So, we will try to **steal this token**.

**How to construct a password reset poisoning attack?**

If the URL sent to the user is generated based on input, such as the Host header, we an construct an attack as follows:

