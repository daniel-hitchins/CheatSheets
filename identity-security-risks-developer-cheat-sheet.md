---
layout: default
title: Identity Security Risks Developer Cheat Sheet
---

# Identity Security Risks Developer Cheat Sheet
## PlatformUI & Mosaic Design System

> For developers building secure Identity management features

## Quick Start: The Golden Rules

1. **Never trust user input** - Always validate and sanitize
2. **Use Mosaic components** - They have built-in protections
3. **Never store secrets in frontend code** - No API keys, tokens, or passwords
4. **Always use HTTPS** - No exceptions in production
5. **Validate on the backend too** - Frontend validation is for UX, not security

---

## Cross-Site Scripting (XSS)

### What It Is

An attacker injects malicious JavaScript into your app that runs in other users' browsers.

### Vue/Mosaic Protection

Vue automatically escapes content, but you can still create vulnerabilities.

**❌ Dangerous:**
```vue
<!-- v-html bypasses Vue's escaping -->
<div v-html="userBio"></div>

<!-- Direct DOM manipulation -->
<script>
document.getElementById('content').innerHTML = userData;
</script>
```

**✅ Safe:**
```vue
<!-- Vue automatically escapes -->
<div>{{ userBio }}</div>

<!-- Mosaic Message component handles escaping -->
<Message severity="info">{{ userMessage }}</Message>
```

### Watch Out For

**User-generated content:**
- Profile descriptions
- Team names
- Application names
- Comments or notes
- Custom labels

**Fix:** Always use `{{ }}` interpolation or Mosaic components. Never use `v-html` with user input.

### URL Parameters

**❌ Dangerous:**
```vue
<script setup>
const route = useRoute();
// XSS if user manipulates URL
document.title = route.query.page;
</script>
```

**✅ Safe:**
```vue
<script setup>
const route = useRoute();
// Vue/Vite handles this safely
useHead({
  title: route.query.page || 'Default Title'
});
</script>
```

---

## Authentication & Authorization

### Token Storage

**❌ Never store in:**
- LocalStorage (accessible to XSS)
- SessionStorage (accessible to XSS)
- JavaScript variables that persist
- URL parameters
- Hidden form fields

**✅ Store in:**
- HTTP-only cookies (set by backend)
- Secure, SameSite cookies
- Memory only (for short-lived tokens)

```vue
<!-- ❌ Bad -->
<script setup>
localStorage.setItem('access_token', token);
</script>

<!-- ✅ Good - let the backend set HTTP-only cookies -->
<script setup>
// Token is in HTTP-only cookie, not accessible to JavaScript
await fetch('/api/login', {
  credentials: 'include' // Sends cookies
});
</script>
```

### Session Management

**Always:**
- Implement session timeout
- Clear sessions on logout
- Re-authenticate for sensitive actions
- Use CSRF tokens for state-changing requests

```vue
<script setup>
const logout = async () => {
  // ✅ Clear everything
  await fetch('/api/logout', { method: 'POST' });
  // Redirect to login
  navigateTo('/login');
};
</script>
```

### Permission Checks

**❌ Frontend checks alone:**
```vue
<!-- This can be bypassed by opening DevTools -->
<Button 
  v-if="user.role === 'admin'" 
  @click="deleteUser"
>
  Delete User
</Button>
```

**✅ Frontend + Backend:**
```vue
<!-- Show button based on permission -->
<Button 
  v-if="user.permissions.includes('users:delete')" 
  @click="deleteUser"
>
  Delete User
</Button>

<script setup>
const deleteUser = async (id) => {
  // Backend verifies permission again
  await api.delete(`/users/${id}`);
};
</script>
```

**Why:** Frontend checks are for UX. Backend must enforce security.

---

## Sensitive Data Exposure

### Personal Identifiable Information (PII)

Never log, display, or expose unnecessarily:
- Passwords (obviously)
- Email addresses (in URLs or analytics)
- Phone numbers
- Social security numbers
- Credit card numbers
- API keys or tokens

**❌ Bad:**
```vue
<script setup>
// Logs user email to browser console
console.log('User logged in:', user.email);

// Shows full email in URL
navigateTo(`/users?email=${user.email}`);
</script>
```

**✅ Good:**
```vue
<script setup>
// Log user ID only
console.log('User logged in:', user.id);

// Use IDs in URLs
navigateTo(`/users/${user.id}`);
</script>
```

### Mask Sensitive Fields

```vue
<!-- ❌ Shows full secret -->
<InputText v-model="apiSecret" />

<!-- ✅ Use Password component for secrets -->
<Password 
  v-model="apiSecret" 
  :feedback="false"
  toggleMask
/>

<!-- ✅ Show masked value with copy button -->
<InputGroup>
  <InputText :value="maskSecret(apiKey)" readonly />
  <Button icon="pi pi-copy" @click="copyToClipboard(apiKey)" />
</InputGroup>

<script setup>
const maskSecret = (secret) => {
  if (!secret) return '';
  return secret.slice(0, 4) + '•'.repeat(secret.length - 8) + secret.slice(-4);
};
</script>
```

### Error Messages

**❌ Reveals too much:**
```vue
<Message severity="error">
  Login failed: User 'john@example.com' not found in database
</Message>
```

**✅ Generic message:**
```vue
<Message severity="error">
  Invalid email or password
</Message>
```

**Why:** Don't confirm which emails are registered. Helps prevent account enumeration.

---

## Injection Attacks

### SQL Injection (Backend Concern)

Your frontend can't prevent this, but don't encourage it:

**❌ Building "queries" in frontend:**
```vue
<script setup>
// Don't do this - encourages bad backend patterns
const filter = `name = '${userName}' AND status = 'active'`;
await api.get(`/users?filter=${filter}`);
</script>
```

**✅ Use structured parameters:**
```vue
<script setup>
const params = {
  name: userName,
  status: 'active'
};
await api.get('/users', { params });
</script>
```

### Command Injection

Never pass user input directly to system commands (this should be caught by backend, but frontend shouldn't enable it).

**❌ Dangerous pattern:**
```vue
<script setup>
// Backend might execute this
await api.post('/files/convert', {
  filename: userFilename,
  options: `--output ${userPath}` // ❌ User controls command
});
</script>
```

**✅ Safe pattern:**
```vue
<script setup>
await api.post('/files/convert', {
  filename: userFilename,
  outputPath: userPath // Backend validates and sanitizes
});
</script>
```

---

## Cross-Site Request Forgery (CSRF)

### What It Is

An attacker tricks a user's browser into making unwanted requests to your app while they're logged in.

### Protection

**Modern frameworks (like Nuxt) often handle this, but verify:**

```vue
<script setup>
// ✅ Use fetch with credentials
await fetch('/api/delete-user', {
  method: 'POST',
  credentials: 'include', // Sends cookies
  headers: {
    'X-CSRF-Token': csrfToken // Your backend provides this
  }
});

// ✅ Or use your API client that handles it
await api.post('/delete-user', data); // If configured correctly
</script>
```

### State-Changing Actions

Always use POST/PUT/DELETE, never GET:

**❌ Dangerous:**
```vue
<!-- Attacker can embed this in an email -->
<a href="/api/delete-account">Click for free stuff!</a>
```

**✅ Safe:**
```vue
<!-- Requires user interaction + CSRF token -->
<Button @click="deleteAccount">Delete Account</Button>

<script setup>
const deleteAccount = async () => {
  await api.post('/delete-account'); // POST with CSRF token
};
</script>
```

---

## File Upload Risks

### Validation

**Always validate:**
- File type (check MIME type AND extension)
- File size
- File name (sanitize special characters)

**❌ Trusting the frontend:**
```vue
<!-- User can bypass this in DevTools -->
<FileUpload accept=".jpg,.png" />
```

**✅ Frontend + Backend validation:**
```vue
<FileUpload 
  accept="image/jpeg,image/png"
  :maxFileSize="2000000"
  @upload="handleUpload"
  aria-describedby="upload-help"
/>
<HelpText id="upload-help">
  JPG or PNG only, max 2MB
</HelpText>

<script setup>
const handleUpload = async (event) => {
  const file = event.files[0];
  
  // ✅ Check again in frontend
  if (!['image/jpeg', 'image/png'].includes(file.type)) {
    toast.add({
      severity: 'error',
      summary: 'Invalid file type',
      detail: 'Only JPG and PNG images are allowed'
    });
    return;
  }
  
  // Backend MUST validate again
  await api.uploadFile(file);
};
</script>
```

### File Names

**❌ Using user's filename directly:**
```vue
<script setup>
// Attacker uploads: ../../../../etc/passwd
const path = `/uploads/${file.name}`;
</script>
```

**✅ Let backend generate safe names:**
```vue
<script setup>
// Backend generates: abc123.jpg
const result = await api.uploadFile(file);
const safePath = result.path; // Backend-controlled
</script>
```

---

## Redirect Vulnerabilities (Open Redirect)

### The Risk

Attacker crafts a URL that redirects users to a malicious site after they interact with your app.

**❌ Dangerous:**
```vue
<script setup>
const route = useRoute();
// Attacker uses: /login?redirect=https://evil.com
const redirectUrl = route.query.redirect;
await login();
window.location = redirectUrl; // ❌ Goes anywhere
</script>
```

**✅ Safe:**
```vue
<script setup>
const route = useRoute();
const redirectUrl = route.query.redirect;

await login();

// ✅ Only allow internal paths
if (redirectUrl && redirectUrl.startsWith('/')) {
  navigateTo(redirectUrl);
} else {
  navigateTo('/dashboard');
}
</script>
```

**Better - Allowlist:**
```vue
<script setup>
const ALLOWED_REDIRECTS = [
  '/dashboard',
  '/profile',
  '/settings'
];

const redirect = route.query.redirect;
const safeRedirect = ALLOWED_REDIRECTS.includes(redirect) 
  ? redirect 
  : '/dashboard';

navigateTo(safeRedirect);
</script>
```

---

## Dependency Vulnerabilities

### Keep Dependencies Updated

```bash
# Check for vulnerabilities
npm audit

# Fix automatically when possible
npm audit fix

# Update Mosaic and other dependencies regularly
npm update @mosaic/design-system
```

### Review Third-Party Libraries

Before adding a dependency:
- Check last update date (active maintenance?)
- Review security advisories
- Check download count (popular = more eyes)
- Read the code if critical functionality

**Red flags:**
- No updates in 2+ years
- Known security issues
- Requests unnecessary permissions
- Obfuscated code

---

## Environment & Configuration

### Environment Variables

**❌ Exposing secrets:**
```vue
<script setup>
// ❌ Bad - API key in frontend code
const API_KEY = 'sk_live_abc123';
</script>
```

**✅ Public variables only:**
```env
# .env
VITE_API_URL=https://api.example.com
VITE_APP_NAME=Identity Manager

# ❌ These get bundled into frontend
# API_SECRET=secret123
```

**Rule:** If it starts with `VITE_` it's public. Only put non-sensitive config there.

### API Endpoints

**Use environment-specific endpoints:**
```vue
<script setup>
const apiUrl = import.meta.env.VITE_API_URL; // From .env

// ✅ Different per environment
// Dev: http://localhost:3000
// Staging: https://staging-api.example.com
// Prod: https://api.example.com
</script>
```

---

## Rate Limiting & DoS Protection

### Prevent Abuse

**Frontend can help:**
```vue
<script setup>
const isSubmitting = ref(false);

const submitForm = async () => {
  if (isSubmitting.value) return; // ✅ Prevent double-submit
  
  isSubmitting.value = true;
  try {
    await api.post('/endpoint', data);
  } finally {
    isSubmitting.value = false;
  }
};
</script>

<template>
  <Button 
    @click="submitForm" 
    :loading="isSubmitting"
    :disabled="isSubmitting"
  >
    Submit
  </Button>
</template>
```

**Use debouncing for search:**
```vue
<script setup>
import { useDebounceFn } from '@vueuse/core';

const search = useDebounceFn(async (query) => {
  await api.get('/search', { params: { q: query } });
}, 300); // ✅ Wait 300ms before searching
</script>
```

**Why:** Reduces server load and prevents accidental DoS from UI interactions.

---

## Browser Security Headers

### Content Security Policy (CSP)

Backend sets these, but frontend must work with them:

```html
<!-- ❌ Inline scripts break CSP -->
<button onclick="alert('hi')">Click</button>

<!-- ✅ Use Vue event handlers -->
<Button @click="handleClick">Click</Button>
```

### Common Headers

Your backend should set:
- `Content-Security-Policy`
- `X-Frame-Options: DENY` (prevents clickjacking)
- `X-Content-Type-Options: nosniff`
- `Strict-Transport-Security` (forces HTTPS)

**Frontend responsibility:** Don't bypass them with workarounds.

---

## Testing for Security Issues

### Manual Testing Checklist

1. **Try to break input validation:**
   - Enter `<script>alert('xss')</script>` in text fields
   - Try SQL injection strings: `' OR '1'='1`
   - Upload non-allowed file types
   - Enter extremely long strings
   - Use special characters: `'; DROP TABLE--`

2. **Check URLs:**
   - Manipulate IDs to access other users' data
   - Remove authentication tokens
   - Try accessing admin pages
   - Test redirect parameters

3. **Inspect network traffic:**
   - Open browser DevTools → Network tab
   - Check for sensitive data in URLs
   - Verify HTTPS is used
   - Check for leaked tokens in requests

4. **Test session management:**
   - Try using expired tokens
   - Test logout functionality
   - Check session timeout
   - Try concurrent sessions

### Automated Tools

- **npm audit** - Dependency vulnerabilities
- **Lighthouse** (Chrome DevTools) - Basic security checks
- **OWASP ZAP** - Web app vulnerability scanner
- **Snyk** - Dependency and code scanning

---

## Common Mistakes to Avoid

1. ❌ Using `v-html` with user input
2. ❌ Storing tokens in localStorage
3. ❌ Only checking permissions in frontend
4. ❌ Logging sensitive data to console
5. ❌ Trusting frontend validation alone
6. ❌ Exposing API keys in code
7. ❌ Using GET requests for state changes
8. ❌ Not sanitizing user filenames
9. ❌ Allowing open redirects
10. ❌ Ignoring `npm audit` warnings
11. ❌ Not validating file uploads properly
12. ❌ Revealing too much in error messages
13. ❌ Not using HTTPS everywhere
14. ❌ Implementing your own crypto (use libraries)

---

## Security by Layer

### Frontend (Your Responsibility)

- Input validation for UX
- XSS prevention (use Vue properly)
- Don't expose secrets
- Sanitize display of user content
- Use Mosaic components (they're tested)

### Backend (Team Responsibility)

- Authentication & authorization enforcement
- Input validation for security
- SQL injection prevention
- Rate limiting
- Session management
- CSRF protection
- Security headers

**Remember:** Frontend security is about defense-in-depth, not being the only line of defense.

---

## Quick Reference: Security Headers

| Header | Purpose | Frontend Impact |
|--------|---------|-----------------|
| `Content-Security-Policy` | Controls resource loading | Blocks inline scripts, restricts domains |
| `X-Frame-Options` | Prevents clickjacking | App can't be embedded in iframes |
| `X-Content-Type-Options` | Prevents MIME sniffing | Files served with correct types |
| `Strict-Transport-Security` | Forces HTTPS | All requests upgrade to HTTPS |
| `X-XSS-Protection` | Browser XSS filter | Legacy protection (CSP is better) |

---

## Resources

- **OWASP Top 10:** https://owasp.org/www-project-top-ten/
- **Vue Security Guide:** https://vuejs.org/guide/best-practices/security.html
- **MDN Web Security:** https://developer.mozilla.org/en-US/docs/Web/Security
- **npm audit docs:** https://docs.npmjs.com/cli/v8/commands/npm-audit

---

## Remember

> "Security is not a feature. It's a practice."

Most security issues are prevented by:
1. Never trusting user input
2. Using Mosaic components (tested and hardened)
3. Validating on both frontend and backend
4. Following the principle of least privilege
5. Keeping dependencies updated

**When in doubt, ask the security team before shipping.**
