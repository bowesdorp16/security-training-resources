# Authentication Security Exercise

## Overview

This hands-on exercise will help you develop skills in identifying and fixing authentication vulnerabilities in a web application using Supabase and Next.js. You'll apply systematic thinking to analyze authentication flows, identify weaknesses, and implement secure alternatives.

## Learning Objectives

By completing this exercise, you will be able to:

- Identify common authentication vulnerabilities
- Implement secure authentication flows
- Conduct methodical security reviews of authentication systems
- Apply defense-in-depth principles to authentication

## Prerequisites

- Basic understanding of Next.js and Supabase
- Completed the Authentication and Authorization module
- Node.js and npm installed
- Git installed

## Exercise Setup

1. Clone the exercise repository:
   ```bash
   git clone https://github.com/bowesdorp16/security-training-resources.git
   cd security-training-resources/exercises/authentication-exercise
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Set up environment variables:
   ```bash
   cp .env.example .env.local
   ```
   
4. Update `.env.local` with your Supabase project credentials

5. Start the development server:
   ```bash
   npm run dev
   ```

## Exercise 1: Vulnerability Identification

### Instructions

1. Review the authentication implementation in the sample application
2. Systematically analyze the following components:
   - User registration process
   - Login implementation
   - Session management
   - Password reset flow
   - Account recovery options

3. Document each vulnerability you find, including:
   - Description of the vulnerability
   - Potential impact of exploitation
   - Steps to reproduce
   - Root cause analysis

### Vulnerabilities to Look For

- Weak password requirements
- Missing email verification
- Insecure session handling
- Vulnerable password reset flow
- Information leakage in error messages
- Missing brute force protection
- Insufficient logging of authentication events

### Expected Output

Create a markdown file named `vulnerabilities.md` with your findings, structured as follows:

```markdown
# Authentication Vulnerabilities Report

## Vulnerability 1: [Brief Title]

**Description:**
Detailed description of the vulnerability

**Severity:** High/Medium/Low

**Impact:**
What could an attacker achieve by exploiting this?

**Steps to Reproduce:**
1. Step 1
2. Step 2
3. etc.

**Root Cause:**
Underlying implementation issue that created this vulnerability

**Recommended Fix:**
Specific recommendations to fix this issue

## Vulnerability 2: [Brief Title]
...
```

## Exercise 2: Secure Implementation

### Instructions

1. Fork the repository to your own GitHub account
2. Create a new branch called `security-fixes`
3. Implement fixes for the vulnerabilities you identified in Exercise 1
4. Write tests to verify your security fixes
5. Document your changes and the security principles applied

### Implementation Requirements

1. **Secure Password Handling:**
   - Implement strong password requirements
   - Add password strength indication
   - Properly store and verify passwords

2. **Email Verification:**
   - Implement a secure email verification flow
   - Prevent sensitive actions until email is verified

3. **Session Security:**
   - Implement secure session management
   - Add proper session expiration
   - Enable session revocation

4. **Secure Password Reset:**
   - Create a secure password reset flow
   - Implement rate limiting for reset attempts
   - Add proper logging and notifications

5. **Brute Force Protection:**
   - Implement account lockout after failed attempts
   - Add CAPTCHA or similar challenge mechanism
   - Implement IP-based rate limiting

### Expected Output

1. Working code with security fixes implemented
2. Test suite demonstrating the security fixes work correctly
3. A `SECURITY.md` file documenting:
   - The vulnerabilities fixed
   - The security controls implemented
   - Any remaining security considerations

## Exercise 3: Security Review

### Instructions

1. Pair up with another participant
2. Exchange implementations for peer review
3. Conduct a thorough security review of their implementation
4. Document your findings and recommendations
5. Discuss your reviews together

### Review Methodology

Follow this systematic approach for your review:

1. **Code Analysis:**
   - Review the authentication implementation line by line
   - Look for security anti-patterns
   - Check for secure coding practices

2. **Test Case Review:**
   - Evaluate test coverage of security features
   - Look for missing edge cases
   - Check test quality and thoroughness

3. **Attack Simulation:**
   - Try to bypass the security controls
   - Test with unexpected inputs
   - Look for logic flaws in the implementation

4. **Documentation Review:**
   - Check completeness of security documentation
   - Verify all vulnerabilities were addressed
   - Look for missing security considerations

### Expected Output

Create a `peer-review.md` file with your findings, including:

1. Overall assessment of the implementation
2. Specific security strengths identified
3. Remaining vulnerabilities or weaknesses
4. Recommendations for further improvements
5. Questions or clarifications needed

## Extension Challenges

If you complete the main exercises, try these extension challenges:

1. **Add Multi-Factor Authentication:**
   - Implement TOTP-based 2FA
   - Add recovery codes mechanism
   - Implement remember-this-device functionality

2. **Advanced Session Management:**
   - Implement active session listing
   - Add ability to terminate individual sessions
   - Create login notifications for new devices

3. **OAuth Integration:**
   - Add secure social login options
   - Implement proper state validation
   - Add account linking functionality

## Submission

1. Push your completed implementation to your GitHub repository
2. Submit the following files:
   - `vulnerabilities.md` from Exercise 1
   - Link to your GitHub repository with fixes
   - `SECURITY.md` from Exercise 2
   - `peer-review.md` from Exercise 3

## Evaluation Criteria

Your submission will be evaluated on:

1. **Completeness:** All vulnerabilities identified and fixed
2. **Correctness:** Security fixes properly implemented
3. **Thoroughness:** Comprehensive testing and documentation
4. **Methodology:** Systematic approach to security analysis
5. **Code Quality:** Clean, maintainable, and secure code

## Additional Resources

- [Supabase Auth Documentation](https://supabase.com/docs/guides/auth)
- [Next.js Authentication Patterns](https://nextjs.org/docs/authentication)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [NIST Digital Identity Guidelines](https://pages.nist.gov/800-63-3/)
