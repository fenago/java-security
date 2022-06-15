
Security Context Basics: User, Subject and Principal 
====================================================

1. Overview 
--------------


Security is a fundamental part of any Java application. Also, we can
find many security frameworks that can handle security concerns.
Additionally, we use a few terms commonly like the subject, principal,
and user in these frameworks.

In this tutorial, we're going to explain these basic concepts of security frameworks. Also, we'll show their relationships and differences.


2. Subject 
-------------


In a security context, the subject represents the source of a request.
[The s]ubject[ is an entity that obtains
information about resources or modifies
resources]. Additionally, a subject can also be
a user, a program, a process, a file, a computer, a database, etc.

For example, a person needs to authorize access to resources and
applications to authenticate the request source. In this case, this
person is a subject.

Let's take a look at our example that implemented base on the JAAS framework:





    Subject subject = loginContext.getSubject();
    PrivilegedAction privilegedAction = new ResourceAction();
    Subject.doAsPrivileged(subject, privilegedAction, null);

3. Principal 
--------------


After successful authentication, we have a populated subject with many associated identities, such as roles, social security number(SSN), etc. In other words, these identifiers are principals, and the subject represents them.

For instance, a person may have an account number principal (“87654-3210”) and other unique identifiers, distinguishing it from other subjects.

Let's see how to create an UserPrincipal after a successful login and add it to a Subject:


    @Override
    public boolean commit() throws LoginException {
        if (!loginSucceeded) {
            return false;
        }
        userPrincipal = new UserPrincipal(username);
        subject.getPrincipals().add(userPrincipal);
        return true;
    }

4. User 
--------


Typically, a user represents a person who accesses resources to perform some action or accomplish a work task.

Also, we can use a user as a principal, and on the other hand, a principal is an identity assigned to a user. UserPrincipal is an excellent example of a user in the JAAS framework discussed in the previous section.





5. Difference Between Subject, Principal, and User 
---------------------------------------------------




As we saw in the above sections, we can represent different aspects of
the same user\'s identity by using principals. They are subsets of
subjects, and users are subsets of principals that are referring to the
end-user or interactive operators.

6. Conclusion 
------------------------------------------------------------------------------------------




In this tutorial, we discussed the definition of the subject, principal,
and user that they are common in most of the security frameworks. Also,
we showed the difference between them.

The implementation of all these examples and code snippets can be found
in [the GitHub project](https://github.com/fenago/java-tutorials/tree/master/core-java-modules/core-java-security-2).

