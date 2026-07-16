Cron is a powerful tool used to schedule recurring tasks (called cron jobs) on Unix-like operating systems, including Linux. Cron jobs are defined in the crontab (cron table) file, and the jobs run at specified intervals, such as hourly, daily, weekly, etc.

## Cron Syntax
Each line in a crontab file follows this structure:

      * * * * * command-to-execute
      │ │ │ │ │
      │ │ │ │ └── Day of the week (0 - 7) (0 or 7 is Sunday)
      │ │ │ └──── Month (1 - 12)
      │ │ └────── Day of the month (1 - 31)
      │ └──────── Hour (0 - 23)
      └────────── Minute (0 - 59)
For example, 15 14 * * 1 /path/to/script.sh will run the script at 14:15 every Monday.

## Special Cron Strings
Cron also allows the use of predefined strings for more readable scheduling:

- @reboot: Runs once at startup.
- @yearly: Runs once a year, equivalent to 0 0 1 1 *.
- @monthly: Runs once a month, equivalent to 0 0 1 * *.
- @weekly: Runs once a week, equivalent to 0 0 * * 0.
- @daily: Runs once a day, equivalent to 0 0 * * *.
- @hourly: Runs once an hour, equivalent to 0 * * * *.

## Setting Up Cron Jobs
### View existing cron jobs:

      crontab -l
      
### Edit cron jobs:


      crontab -e
      
### Remove cron jobs:


      crontab -r
      
## Log Output
To log output from a cron job to a file:


      * * * * * /path/to/script.sh >> /var/log/mylogfile.log 2>&1
This ensures that both standard output and errors are written to the log file.
