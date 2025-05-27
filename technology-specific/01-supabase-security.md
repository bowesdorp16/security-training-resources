# Supabase Security

## Overview

Supabase provides a powerful set of tools for building secure applications, but requires proper configuration and understanding to prevent security vulnerabilities. This module covers security best practices specific to Supabase, with a focus on Row Level Security (RLS), API key management, secure authentication flows, and data access policies.

## Learning Objectives

By the end of this module, you should be able to:

- Configure and implement Row Level Security (RLS) policies
- Properly manage and secure API keys in Supabase projects
- Implement secure authentication flows using Supabase Auth
- Design and enforce effective data access policies
- Identify and mitigate common Supabase-specific security vulnerabilities

## Row Level Security (RLS)

### What is RLS?

Row Level Security is a PostgreSQL feature that allows you to define security policies that restrict which rows users can access in a database table. Supabase makes RLS easy to implement, but requires careful design to ensure proper security.

### Common RLS Vulnerabilities

1. **Missing RLS Policies**
   - Tables without RLS enabled are publicly accessible
   - Default behavior allows full access if no policies exist

2. **Overly Permissive Policies**
   - Policies that don't properly check user identity
   - Using the `true` condition without proper filtering

3. **Incomplete Policy Coverage**
   - Securing some operations (SELECT) but not others (INSERT, UPDATE, DELETE)
   - Not accounting for all access patterns

4. **Logic Errors in Policies**
   - Incorrect boolean logic in policy definitions
   - Misunderstanding policy combination (OR vs. AND)

5. **Leaking Data Through Joins**
   - Unsecured tables accessible through joins from secured tables
   - Not considering how views interact with RLS

### Best Practices

1. **Enable RLS on All Tables**
   ```sql
   -- Always enable RLS on tables with sensitive data
   ALTER TABLE my_table ENABLE ROW LEVEL SECURITY;
   ```

2. **Default to Deny**
   - Start with no access and explicitly grant permissions
   - Never use `true` without careful consideration

3. **Write Specific Policies**
   ```sql
   -- Example: Users can only see their own data
   CREATE POLICY "Users can view own data" ON profiles
     FOR SELECT
     USING (auth.uid() = user_id);
   ```

4. **Define Policies for Each Operation**
   ```sql
   -- Separate policies for different operations
   CREATE POLICY "Users can update own data" ON profiles
     FOR UPDATE
     USING (auth.uid() = user_id);
   ```

5. **Test Policies Thoroughly**
   - Test from different user contexts
   - Verify each operation type (SELECT, INSERT, UPDATE, DELETE)
   - Check edge cases and boundary conditions

### Real-World Example

```sql
-- Table definition
CREATE TABLE projects (  
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  description TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  owner_id UUID NOT NULL REFERENCES auth.users(id),
  is_public BOOLEAN DEFAULT FALSE
);

-- Enable RLS
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;

-- Policy for viewing projects
CREATE POLICY "Users can view own projects" ON projects
  FOR SELECT
  USING (
    auth.uid() = owner_id -- User owns the project
    OR is_public = TRUE    -- OR project is public
  );

-- Policy for inserting projects
CREATE POLICY "Users can create own projects" ON projects
  FOR INSERT
  WITH CHECK (auth.uid() = owner_id);

-- Policy for updating projects
CREATE POLICY "Users can update own projects" ON projects
  FOR UPDATE
  USING (auth.uid() = owner_id);

-- Policy for deleting projects
CREATE POLICY "Users can delete own projects" ON projects
  FOR DELETE
  USING (auth.uid() = owner_id);
```

## API Key Management

### Types of Supabase API Keys

1. **anon key** (public) - Used for unauthenticated operations
2. **service_role key** (private) - Bypasses RLS with admin privileges

### Common API Key Vulnerabilities

1. **Exposing service_role Key**
   - Including in client-side code
   - Storing in unsecured configuration
   - Committing to public repositories

2. **Overly Permissive anon Key Settings**
   - Not restricting what unauthenticated users can access
   - Relying solely on client-side checks

3. **No Key Rotation Strategy**
   - Using the same keys indefinitely
   - No process for key compromise response

4. **Insecure Key Storage**
   - Hardcoding keys in application code
   - Storing in unencrypted configuration files

### Best Practices

1. **Never Expose service_role Key**
   - Only use in server-side code or secure environments
   - Use environment variables for storage

2. **Properly Secure the anon Key**
   ```javascript
   // Client-side code should only use the anon key
   const supabase = createClient(
     'https://your-project.supabase.co',
     'your-anon-key'
   );
   ```

3. **Implement Proper RLS for anon Key Access**
   ```sql
   -- Example: Allow anyone to read public data
   CREATE POLICY "Public profiles are viewable by everyone" 
     ON profiles FOR SELECT 
     USING (is_public = TRUE);
   ```

4. **Server-side Operations with service_role**
   ```javascript
   // Server-side code can use the service_role key when needed
   const adminSupabase = createClient(
     process.env.SUPABASE_URL,
     process.env.SUPABASE_SERVICE_KEY
   );
   ```

5. **Use Environment Variables**
   ```javascript
   // Next.js example
   // In .env.local (not committed to repo)
   NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
   NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
   SUPABASE_SERVICE_KEY=your-service-role-key
   
   // In code
   const supabase = createClient(
     process.env.NEXT_PUBLIC_SUPABASE_URL,
     process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY
   );
   ```

## Secure Authentication Flows

### Authentication Methods in Supabase

1. Email/Password
2. Magic Link
3. OAuth Providers
4. Phone Auth

### Common Authentication Vulnerabilities

1. **Insecure Email Verification**
   - Not requiring email verification
   - Weak verification token generation

2. **Password Reset Flaws**
   - Vulnerable password reset flows
   - Lacking rate limiting on reset attempts

3. **OAuth Configuration Issues**
   - Incorrect redirect URI configuration
   - Insecure state parameter handling

4. **Session Management Weaknesses**
   - Overly long session durations
   - No session revocation capability

### Best Practices

1. **Implement Complete Authentication Flows**
   ```javascript
   // Sign up with email verification
   const signUp = async (email, password) => {
     const { user, error } = await supabase.auth.signUp({
       email,
       password,
     }, {
       // Redirect to confirmation page
       redirectTo: 'https://example.com/auth/confirm',
     });
     
     // Handle error and success cases
   };
   ```

2. **Secure Password Reset**
   ```javascript
   // Request password reset
   const resetPassword = async (email) => {
     const { data, error } = await supabase.auth.resetPasswordForEmail(email, {
       redirectTo: 'https://example.com/auth/update-password',
     });
     
     // Handle error and success cases
   };
   ```

3. **Proper OAuth Configuration**
   - Configure correct redirect URIs in provider settings
   - Implement proper state validation
   - Secure storage of provider credentials

4. **Session Security**
   ```javascript
   // Configure shorter session duration
   const { data, error } = await supabase.auth.signIn(
     { email, password },
     { expiresIn: 3600 } // 1 hour in seconds
   );
   
   // Sign out from all devices
   const { error } = await supabase.auth.signOut({
     scope: 'global'
   });
   ```

5. **Multiple Authentication Factors**
   - Implement multi-factor authentication when possible
   - Consider using adaptive authentication based on risk

## Data Access Policies

### Types of Data Access Controls

1. Row Level Security (RLS)
2. Column-level Permissions
3. Function-based Access Control
4. Database Roles and Privileges

### Common Access Policy Vulnerabilities

1. **Insufficient Access Granularity**
   - All-or-nothing access controls
   - Lack of field-level security

2. **Data Leakage Through Related Tables**
   - Secured primary tables but unsecured related data
   - Not considering data relationships in policy design

3. **Bypassing Policies Through Functions**
   - Insecure functions that reveal protected data
   - Not applying proper security contexts to functions

4. **Temporal Access Issues**
   - Not handling time-based access restrictions
   - Permanent access instead of time-limited grants

### Best Practices

1. **Design Data Access by User Role**
   ```sql
   -- Example: Different access for different roles
   CREATE POLICY "Admins have full access" ON documents
     USING (EXISTS (
       SELECT 1 FROM user_roles
       WHERE user_roles.user_id = auth.uid()
       AND user_roles.role = 'admin'
     ));
   ```

2. **Secure Related Data**
   ```sql
   -- Ensure both tables are properly secured
   CREATE POLICY "Users can access their project comments" 
     ON project_comments
     FOR SELECT
     USING (
       EXISTS (
         SELECT 1 FROM projects 
         WHERE projects.id = project_comments.project_id
         AND projects.owner_id = auth.uid()
       )
     );
   ```

3. **Implement Attribute-Based Access Control**
   ```sql
   -- Example: Access based on document attributes
   CREATE POLICY "Access by document status" ON documents
     FOR SELECT
     USING (
       CASE
         WHEN auth.uid() = documents.owner_id THEN TRUE
         WHEN documents.status = 'public' THEN TRUE
         WHEN documents.status = 'internal' AND 
              EXISTS (SELECT 1 FROM employees WHERE user_id = auth.uid()) THEN TRUE
         ELSE FALSE
       END
     );
   ```

4. **Time-Limited Access**
   ```sql
   -- Example: Time-limited access to resources
   CREATE POLICY "Temporary access" ON shared_documents
     FOR SELECT
     USING (
       auth.uid() = shared_documents.shared_with_user_id
       AND shared_documents.expires_at > now()
     );
   ```

5. **Function Security with Security Definer**
   ```sql
   -- Secure function that enforces access control
   CREATE OR REPLACE FUNCTION get_document(document_id UUID)
   RETURNS SETOF documents
   LANGUAGE sql
   SECURITY DEFINER
   SET search_path = public
   AS $$
     SELECT * FROM documents 
     WHERE id = document_id 
     AND (owner_id = auth.uid() OR is_public = TRUE);
   $$;
   ```

## Security Testing for Supabase

### Testing Approaches

1. **RLS Policy Testing**
   - Test each policy with different user contexts
   - Verify policy combinations work as expected
   - Test edge cases and boundary conditions

2. **API Security Testing**
   - Test API endpoints with different authentication states
   - Verify proper key usage and permissions
   - Test for information leakage

3. **Authentication Flow Testing**
   - Test complete authentication flows
   - Verify email verification and password reset
   - Test session management and expiration

4. **Data Access Testing**
   - Verify proper data isolation between users
   - Test access to related data
   - Check temporal access restrictions

### Automated Testing Example

```javascript
// Jest test example for RLS policies
describe('RLS Policies', () => {
  let userAClient, userBClient, anonClient, adminClient;
  
  beforeAll(async () => {
    // Set up test clients with different authentication contexts
    adminClient = createClient(process.env.SUPABASE_URL, process.env.SUPABASE_SERVICE_KEY);
    anonClient = createClient(process.env.SUPABASE_URL, process.env.SUPABASE_ANON_KEY);
    
    // Create test users
    const { data: userA } = await adminClient.auth.admin.createUser({
      email: 'usera@example.com',
      password: 'securepassword',
      email_confirm: true
    });
    
    const { data: userB } = await adminClient.auth.admin.createUser({
      email: 'userb@example.com',
      password: 'securepassword',
      email_confirm: true
    });
    
    // Create authenticated clients
    const { data: userAAuth } = await supabase.auth.signInWithPassword({
      email: 'usera@example.com',
      password: 'securepassword',
    });
    userAClient = createClient(
      process.env.SUPABASE_URL,
      process.env.SUPABASE_ANON_KEY,
      { global: { headers: { Authorization: `Bearer ${userAAuth.session.access_token}` } } }
    );
    
    // Repeat for userB
    // ...
    
    // Create test data
    await adminClient.from('projects').insert([
      { name: 'User A Project', owner_id: userA.id, is_public: false },
      { name: 'User B Project', owner_id: userB.id, is_public: false },
      { name: 'Public Project', owner_id: userA.id, is_public: true }
    ]);
  });
  
  test('Users can only see their own private projects', async () => {
    // User A should see only their private project and the public project
    const { data: userAProjects, error: userAError } = await userAClient
      .from('projects')
      .select('*')
      .eq('is_public', false);
      
    expect(userAError).toBeNull();
    expect(userAProjects.length).toBe(1);
    expect(userAProjects[0].name).toBe('User A Project');
    
    // User B should not see User A's private projects
    const { data: userBProjects, error: userBError } = await userBClient
      .from('projects')
      .select('*')
      .eq('owner_id', userA.id)
      .eq('is_public', false);
      
    expect(userBError).toBeNull();
    expect(userBProjects.length).toBe(0);
  });
  
  test('Anyone can see public projects', async () => {
    // Anonymous client should see only public projects
    const { data: anonProjects, error: anonError } = await anonClient
      .from('projects')
      .select('*')
      .eq('is_public', true);
      
    expect(anonError).toBeNull();
    expect(anonProjects.length).toBe(1);
    expect(anonProjects[0].name).toBe('Public Project');
  });
  
  // More tests for insert, update, delete operations
  // ...
});
```

## Exercises

1. **RLS Policy Design**
   - Design RLS policies for a multi-tenant application
   - Implement and test policies for different user types
   - Document the security rationale for each policy

2. **Authentication Flow Implementation**
   - Implement a complete authentication flow with Supabase
   - Include email verification and password reset
   - Add multi-factor authentication

3. **Security Audit**
   - Audit an existing Supabase project for security issues
   - Identify vulnerabilities in RLS policies and authentication
   - Create a remediation plan for identified issues

## Additional Resources

- [Supabase Security Documentation](https://supabase.io/docs/guides/auth/row-level-security)
- [PostgreSQL RLS Documentation](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)
- [Supabase Auth Helpers](https://github.com/supabase/auth-helpers)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)

## Next Steps

After completing this module, proceed to the following related topics:

1. Next.js Security Best Practices
2. API Security Implementation
3. Data Storage Security
4. Security Testing and Verification
