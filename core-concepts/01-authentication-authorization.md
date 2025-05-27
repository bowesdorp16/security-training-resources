# Authentication and Authorization

## Overview

Authentication and authorization are the foundation of web application security. Understanding these concepts thoroughly and implementing them correctly is critical to protecting user data and system resources.

## Learning Objectives

By the end of this module, you should be able to:

- Explain the difference between authentication and authorization
- Identify common vulnerabilities in authentication systems
- Implement secure authentication using industry best practices
- Design effective authorization models
- Conduct systematic security reviews of auth systems

## Authentication

### Definition

Authentication is the process of verifying the identity of a user, system, or entity. It answers the question: "Who are you?"

### Common Vulnerabilities

1. **Weak Password Policies**
   - Allows easily guessable passwords
   - No enforcement of complexity requirements
   - No protection against common passwords

2. **Brute Force Attacks**
   - No rate limiting
   - No account lockout mechanisms
   - No detection of repeated failed attempts

3. **Credential Storage Issues**
   - Plaintext password storage
   - Weak hashing algorithms
   - No salting of passwords

4. **Session Management Flaws**
   - Insecure session identifiers
   - Long-lived sessions
   - No proper session invalidation

5. **Insecure Communication**
   - No TLS/SSL for login pages
   - Mixed content issues
   - Weak cipher suites

### Best Practices

1. **Strong Password Policies**
   - Minimum length requirements (12+ characters recommended)
   - Complexity requirements (mix of character types)
   - Password strength meters
   - Check against lists of compromised passwords

2. **Multi-Factor Authentication (MFA)**
   - Something you know (password)
   - Something you have (security key, phone)
   - Something you are (biometrics)
   - Implementation considerations for each factor

3. **Secure Credential Storage**
   - Use modern hashing algorithms (Argon2, bcrypt, PBKDF2)
   - Proper salt generation and storage
   - Work factor adjustment over time

4. **Robust Session Management**
   - Secure, random session identifiers
   - Appropriate session timeouts
   - Session regeneration after privilege changes
   - Proper session invalidation on logout

5. **Secure Communication**
   - TLS for all authentication traffic
   - Proper certificate validation
   - HTTP Strict Transport Security (HSTS)

## Authorization

### Definition

Authorization is the process of determining whether an authenticated entity has permission to access a resource or perform an action. It answers the question: "What are you allowed to do?"

### Common Vulnerabilities

1. **Insecure Direct Object References (IDOR)**
   - No validation of resource ownership
   - Predictable resource identifiers
   - Lack of access control checks

2. **Missing Function Level Access Control**
   - Security by obscurity
   - Client-side only access controls
   - Lack of consistent authorization checks

3. **Privilege Escalation**
   - Horizontal privilege escalation (accessing other users' data)
   - Vertical privilege escalation (gaining higher privileges)
   - Role confusion issues

4. **Overprivileged Access**
   - Not following principle of least privilege
   - Excessive permissions for service accounts
   - Lack of permission granularity

### Best Practices

1. **Role-Based Access Control (RBAC)**
   - Clearly defined roles and permissions
   - Hierarchical role structures when appropriate
   - Regular review of role assignments

2. **Attribute-Based Access Control (ABAC)**
   - Context-aware authorization decisions
   - Policy-based approaches
   - Dynamic adjustment based on risk factors

3. **Principle of Least Privilege**
   - Grant minimal necessary permissions
   - Time-bound access when possible
   - Regular permission reviews

4. **Centralized Authorization**
   - Consistent enforcement points
   - Reusable authorization components
   - Comprehensive logging of access decisions

5. **Defense in Depth**
   - Multiple layers of access controls
   - Independent verification at different levels
   - No assumptions about upstream security controls

## Implementation Examples

### Secure Authentication with Supabase

```javascript
// Example of secure authentication implementation using Supabase
const signUp = async (email, password) => {
  try {
    // Validate input before sending to server
    if (!isValidEmail(email) || !isStrongPassword(password)) {
      throw new Error('Invalid email or password requirements not met');
    }
    
    // Use Supabase's auth services with proper error handling
    const { user, error } = await supabase.auth.signUp({
      email,
      password,
    });
    
    if (error) throw error;
    
    // Handle successful signup
    return { success: true, user };
  } catch (error) {
    // Proper error handling without revealing sensitive information
    console.error('Signup error:', error);
    return { 
      success: false, 
      message: 'Account creation failed. Please try again.' 
    };
  }
};

// Helper function to validate email format
const isValidEmail = (email) => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
};

// Helper function to check password strength
const isStrongPassword = (password) => {
  // Check length
  if (password.length < 12) return false;
  
  // Check for variety of character types
  const hasUppercase = /[A-Z]/.test(password);
  const hasLowercase = /[a-z]/.test(password);
  const hasNumbers = /\d/.test(password);
  const hasSpecialChars = /[!@#$%^&*(),.?":{}|<>]/.test(password);
  
  // Require at least 3 of the 4 character types
  const conditionsMet = [hasUppercase, hasLowercase, hasNumbers, hasSpecialChars]
    .filter(Boolean).length;
  
  return conditionsMet >= 3;
};
```

### Role-Based Access Control in Next.js API Routes

```javascript
// Example of RBAC middleware for Next.js API routes
import { getSession } from 'next-auth/react';

// Middleware to check for required roles
export function withRoleCheck(handler, requiredRoles = []) {
  return async (req, res) => {
    // Get user session
    const session = await getSession({ req });
    
    // Check if user is authenticated
    if (!session || !session.user) {
      return res.status(401).json({ 
        error: 'You must be signed in to access this resource.' 
      });
    }
    
    // If no specific roles are required, just need to be authenticated
    if (requiredRoles.length === 0) {
      return handler(req, res);
    }
    
    // Check if user has any of the required roles
    const userRoles = session.user.roles || [];
    const hasRequiredRole = requiredRoles.some(role => 
      userRoles.includes(role)
    );
    
    if (!hasRequiredRole) {
      return res.status(403).json({ 
        error: 'You do not have permission to access this resource.' 
      });
    }
    
    // User has required role, proceed to handler
    return handler(req, res);
  };
}

// Example usage in an API route
export default withRoleCheck(async (req, res) => {
  // Only users with 'admin' role can access this endpoint
  try {
    const data = await fetchSensitiveData();
    res.status(200).json(data);
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch data' });
  }
}, ['admin']);
```

## Systematic Security Review

When reviewing authentication and authorization systems, follow this methodical approach:

1. **Map the Authentication Flow**
   - Identify all entry points and authentication mechanisms
   - Document the credential handling process
   - Analyze session management implementation

2. **Identify Authorization Checkpoints**
   - Map all protected resources and actions
   - Document how access decisions are made
   - Verify consistency across similar resources

3. **Test Boundary Conditions**
   - Try to access resources without authentication
   - Test with expired or invalid credentials
   - Attempt to bypass authentication steps

4. **Verify Authorization Logic**
   - Test access with different user roles
   - Attempt horizontal and vertical privilege escalation
   - Verify direct object reference protection

5. **Review Error Handling**
   - Check for information leakage in error messages
   - Verify consistent error responses
   - Test error handling for edge cases

## Exercises

1. **Authentication System Review**
   - Analyze a sample authentication system
   - Identify at least five potential vulnerabilities
   - Suggest specific improvements for each issue

2. **Authorization Matrix Development**
   - Create a comprehensive role-permission matrix for a sample application
   - Document the rationale for each permission assignment
   - Identify potential edge cases in the authorization model

3. **Implementation Challenge**
   - Implement a secure authentication system using Supabase
   - Add multi-factor authentication support
   - Create a role-based access control system for protected resources

## Additional Resources

- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
- [NIST Digital Identity Guidelines](https://pages.nist.gov/800-63-3/)
- [Auth0 Identity Management Best Practices](https://auth0.com/blog/identity-management-best-practices/)
- [Supabase Auth Documentation](https://supabase.io/docs/guides/auth)

## Next Steps

After completing this module, proceed to the following related topics:

1. Session Management Security
2. JWT Security Best Practices
3. OAuth and OpenID Connect Implementation
4. API Security
