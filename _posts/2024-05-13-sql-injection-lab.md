---
layout: distill
title: SQL Injection Lab - Altoro Mutual
description: this lab was a part of my Ethical Hacking and Penetration Testing course at SUNY Canton
tags: tech hacking lab
giscus_comments: true
date: 2024-05-13
featured: true

authors:
  - name: Colin Toomey
    url: "https://crtoomey.github.io/"
    affiliations:
      name: 

bibliography:

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).
toc:
  - name: Disclaimer
    # if a section has subsections, you can add them as follows:
    # subsections:
    #   - name: Example Child Subsection 1
    #   - name: Example Child Subsection 2
  - name: Introduction
  - name: First Approach
  - name: Setbacks
  - name: Character by Character
  - name: Lessons Learned
  - name: References

---

## Disclaimer

This lab was conducted for educational purposes as part of the Ethical Hacking and Penetration Testing course at SUNY Canton, using a deliberately vulnerable website designed for training. The focus of this lab is on SQL injection. The methods and techniques discussed in this blog post are intended solely for educational use and should not be applied to engage in illegal activities. Always obtain explicit permission before conducting penetration testing on any system.

## Introduction

For this ethical hacking lab, I was instructed to find 10 pairs of usernames and passwords to login into [Altoro Mutual](http://www.testfire.net/), a website that was designed to be vulnerable, using SQL injection (SQLi). This blog post details the approaches, methods, and process of completing this lab and also my thoughts on how to mitigate SQL injection vulnerabilities. I used a variety of resources that were either given to me by my professor or that I found on the web. Those resources can be found in the References section of this blog post. I will also cover what my professor was actually expecting us to do at the end, which will hopefully be as funny to you as it was for me when he told us. 

## First Approach

I first injected the SQL statement **'OR''='** for both the username and password inputs which bypassed the login. This method logged me in as the admin user which had a link to a page called "Edit Users". See Image 1. After navigating to this page, I was presented with a few options like adding new members and changing the passwords for existing users. See Image 2. However, none of these features actually worked so I was going to have to do it the hard way. At this point in the lab, I had 5 usernames and a way to bypass the login authentication but no passwords.  

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Altoro-Mutual-Image-1" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Altoro-Mutual-Image-2" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Admin access (Image 1) and a list of users existing in the website's database (Image 2).
</div>

## Setbacks

Following the login bypass, I attempted to enumerate the table that held the login credentials of users with SQL injections like **A' OR SELECT * FROM users; --** but I didn't find too much information about the table. At this point, I tried some Kali Linux tools used for SQL injection, specifically sqlmap. I tried to get sqlmap to work properly, but, with the time constraints I had, I couldn't spend the time to properly use it. I tried other tools like BurpSuite but that didn't lead to new information either. 

After the failure to enumerate the table name and information about the table, I spent quite a bit of time looking around the website. I found that the "Customize Site Language" page was vulnerable to cross-site scripting (XSS) but this was also a dead end for me. See Image 3. I was bit defeated at this point so I decided to take a break and come back the next day.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Altoro-Mutual-Image-3" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    XSS on "Customize Site Language" Page (Image 3).
</div>

## Progress

After the break I took, I felt like I had to go back to square one as nothing seemed to work. I attempted to login with the SQL statement **test' OR 1=1;--** in the username input. I did a variety of these and nothing worked which seemed odd to me. I knew that **'OR''='** worked in both the username and password inputs, so I knew that my SQL statements haven’t been in the proper form. With this in mind, I tried **test'OR''='** in both the username and password inputs and was able to login.

Because of this revelation, I will try to figure out the SQL statement that is used for the username and password based on my successful login.
**SELECT * FROM tablename WHERE tablecolumn = ' ' OR ' ' = ' '**
**SELECT * FROM tablename WHERE tablecolumn = 'test' OR ' ' = ' '**
So the basic idea of this specific SQL injection is saying this blank input or nothing equals nothing, which evaluates to true and bypasses authentication. So, now that I have a way to test true or false values, I will use the [Unix Wiz method](http://www.unixwiz.net/techtips/sql-injection.html) for mapping out the table columns. I ran a lot of tests with statements similar to **Test' AND uid IS NULL; --** but got errors everytime. Eventually, I tried **Test' OR Accounts IS NULL --** and got a more interesting error. See Image 4.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Altoro-Mutual-Image-4" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    New Error Saying Accounts is Not in the Table (Image 4).
</div>

Now I had a SQL injection that would allow me to test true or false statements. I tested to see if *UID* was a column and if *Passw* was a column. I tried some variations on both until no error is thrown. After a few iterations for the username column, I used User_ID and got no error so User_ID was one of the columns in the table. Password was also in the table. First_name was a column and so was last_name. I tried a few others like email and variations on accounts but nothing seemed to work. Now, using the Unix Wiz method we will try to find the table name.

To find the table name I used the SQL injection **Test' OR 1=(SELECT COUNT(*) FROM users) --** and got a new error. See Image 5. With this injection I could find the name of the table. I had a few guesses and finally got no errors when I used "People" instead of "users" in the SQL injection at the start of this paragraph. Now I wanted to try table.field notation just to make sure that people is indeed the table name. The statement **Test' OR People.User_ID IS NULL --** does not give any errors but **Test' OR Members.User_ID IS NULL --** does so the table name seems to be people. However, at this point, I ran into another problem which was I had no way of displaying the usernames and passwords. I tried a variety of things like UNION attacks but had no luck.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Altoro-Mutual-Image-5" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Error Saying Table "Users" Does Not Exist (Image 5).
</div>


## Character by Character

After the setbacks during enumeration and my failure to get the usernames and passwords, I studied up on SQL a bit more as I was a bit rusty in my knowledge of it. After some research, I realized I could use the LIKE keyword with SQL to get each character of each user’s password. Using this SQL injection **A' OR User_ID LIKE 'sspeed' --** I got an HTTP Status 500 error. See Image 6. However, when I used this injection **A' OR User_ID LIKE 'borg' --** I got the normal username or password doesn’t exist error. So I tried to use this method with wildcards to see if I could get the passwords character by character. If that didn't work I planned to do a dictionary attack with Hydra on Kali Linux.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Altoro-Mutual-Image-6" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Internal Server Error (Image 6).
</div>

To test this method, I checked to see if I get the internal server error with a user_id I know starts with ‘s’ (sspeed) and use A' OR User_ID LIKE 's%' --. I got the internal server error and tried ‘z%’ which did not give me the internal server error. Now, I knew this method should theoretically work for passwords so I tried **A' OR Password LIKE 'a%' --**. This gave me the internal server error but ‘z%’ did not. Now, combining wildcards to check for the length of the password starting with ‘a’ using the injection **A' OR Password LIKE 'a_%' --**. I did this until I got the internal server error and note how long the password is. This immediately gave me an internal server error which seems wrong unless someone actually has a 2 character password that starts with "a" so I try to use **A' OR Password LIKE 'a_' --**. This doesn’t give me an internal server error so it should work. I iterated through the password length adding one _ every time until I got to 'a____' or 'a' followed by 4 characters. I knew that a password starts with 'a' and is five characters long. I could now iterate through each character until I can guess the password. I got an internal server error at ‘ad___’ and I think I have a guess as to what this password is. I try ‘adm__’ and again, server error so I keep going until I get 'admin'. 'Admin' is a password for an account but which account/accounts? Obviously, the first guess is the admin user so I attempted the login with admin as username and password. It worked. I checked this password against all other users to make sure it is only for the admin account and it was.

I did this same method for all of the other user's passwords and found a total of 3 passwords for 5 users. The username and password pairs are:

  1.	Admin, admin
  2.	Jsmith, demo1234
  3.	Jdoe, demo1234
  4.	Sspeed, demo1234
  5.	Tuser, tuser

The instructions for this lab asked for 10 username and password pairs so I tried to enumerate other usernames using the same method I did (character by character) that weren't shown in the "Edit Users" page but I couldn't find any other users. I had spent ~40 hours across 6 days so I was satisfied with my effort and was okay with getting points taken off the assignment for not finding 10 usernames and passwords. Plus, in the real world, bypassing authentication and gaining admin access to the website would have allowed me to do serious damage to the website, steal data, or do a host of other things. 


## Lessons Learned

Now, I would like to share some of the lessons I learned and some of the lessons you should take away from this lab write-up. The first lesson I learned is that hacking tools are not a quick fix for lacking knowledge and often require users to have a high level understanding of what the tool is doing. The second is, in hacking, you will fail frequently and get frustrated but that *is* okay. In fact, if I had immediately succeeded finding the usernames and passwords using a tool like sqlmap, I would have learned absolutely nothing. The third lesson I learned is that hacking is done by cheaters and is not fair. I say this because the GitHub repository for Altoro Mutual was public and I could have just used that to find the usernames and passwords (they were in a file that had code that setup the database). 

So how do we stop SQL injections? That is the point of this exercise after all, we want to be able to defend against attacks like these. There's a few methods an organization could use to stop a SQL injection attack. First, an organization could use prepared statements to take the input, say **test' or '1'='1**, and look for a username that matched exactly that entire string (OWASP). Many programming languages have functions that allow programmers to implement prepared statements. Next, you could use stored procedures, although these are not always a great option and can be still susceptible to SQL injection. Stored procedures are similar to prepared statements but are "stored in the database itself, and then called from the application" (OWASP). An organization or individual could also use allow-list input validation where the application ensures that data entering the system is properly formed and any bad or injected data is prevented from entering the database (OWASP). You could also escape all user input, however, this is strongly discouraged by the Open Worldwide Application Security Project (OWASP) because it is a weak countermeasure compared to other mitigation methods. If you would like to see more in depth information about how to protect against SQL injections, visit the [SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html) that was made by OWASP.

I promised I would tell you what my professor was actually expecting us as students to do and I will right now. The correct answer to this lab was:

  1. **' OR '' = '**
  2. **' OR '1' = '1**
  3. **' OR '2' = '2**
  4. **' OR '3' = '3**
  5. **' OR '4' = '4**
  6. **' OR '5' = '5**
  7. **' OR '6' = '6**
  8. **' OR '7' = '7**
  9. **' OR '8' = '8**
  10. **' OR '9' = '9**

Or anything that allowed for authentication bypass. I'll be honest when my professor told us this, I actually laughed out loud because what else can you do in that situation?

Thanks for reading.

## References

- https://www.w3schools.com/sql/sql_injection.asp
- http://www.unixwiz.net/techtips/sql-injection.html 
- https://www.kali.org/tools/sqlmap/
- https://www.geeksforgeeks.org/use-sqlmap-test-website-sql-injection-vulnerability/
- https://book.hacktricks.xyz/pentesting-web/sql-injection/sqlmap
- https://www.w3schools.com/sql/sql_wildcards.asp 
- https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html