## "Kortenberg": Can't touch this!

### Problem

Description: Is "All I want for Christmas is you" already everywhere?. A bit unrelated, someone messed up the permissions in this server, the admin user can't list new directories and can't write into new files. Fix the issue.
NOTE: Besides solving the problem in your current admin shell session, you need to fix it permanently, as in a new login shell for user "admin" (like the one initiated by the scenario checker) should have the problem fixed as well.

Root (sudo) Access: True

Test: The admin user in a separate Bash login session should be able to create a new directory in your /home/admin directory, as well as being able to create a file into this new directory and add text into the new file.

### Solution

