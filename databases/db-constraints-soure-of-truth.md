# Database Constraints Are the Source of Truth

Application-level checks are not sufficient and can break
when concurrent requests happen.

For example:
Request A -> checks if class exists
Request B -> checks if class exists

Request A -> INSERT class
Request B -> INSERT class

Now you have **duplicate classes**
This is a classic race condition. A DB constraint would prevent 
such scenarios and fail either Request A or Requst B

## Rule of Thumb
If a business rule must always hold true, enfore it at the DB level too