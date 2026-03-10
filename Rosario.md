# Rosario


Description: A developer created a database named 'main' but now some data is missing in the database. You need to restore the database using the the dump "/home/admin/backup.sql".
The issue is that the developer forgot the root password for the MariaDB server.
If you encounter an issue while restoring the database, fix it.


We have root privelged so it wouldnt be any hard to bypass this mariadb password.

Found a quick guide : https://www.digitalocean.com/community/tutorials/how-to-reset-your-mysql-or-mariadb-root-password

it started MariaDB without the Grant tables which seemed to get rid of the password.

now it was just about restoring the dump.


however,

```
MariaDB [main]> source /home/admin/backup.sql ERROR 1064 (42000) at line 7 in file: '/home/admin/backup.sql': You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near '? /*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */?
```


On analysing the dump. I found all the ; had been turned to ? maybe unrecognised character or smthing idk.

Just went into vim a quick : `%s/?/;/g`

and we are done. Just restore the dump with a <

GGs




