#### Setting Google Cloud Storage Bucket and File permissions.
* Google Cloud SQL requires you to set writer permissions on the bucket that contains your database dump file.
  ```
  gsutil acl ch -u <service_account_associated_with_sql_instance>:W gs://<bucket_name>
  ```
  Example:
  ```
  gsutil acl ch -u adnrtynbqzayvhpovum46waf64@speckle-umbrella-38.iam.gserviceaccount.com:W gs://bucket-22062018-104711
  ```
  * In above example:
    ```
    * service_account_associated_with_sql_instance = adnrtynbqzayvhpovum46waf64@speckle-umbrella-38.iam.gserviceaccount.com
    * bucket_name = bucket-22062018-104711
    ```

* Google Cloud requires you to set reader permissions on the dump file that you are going to use to import data into your database
  ```
  gsutil acl ch -u <service_account_associated_with_sql_instance>:R gs://<bucket_name>/<file_name_with_extension>
  ```
  Example
  ```
  gsutil acl ch -u adnrtynbqzayvhpovum46waf64@speckle-umbrella-38.iam.gserviceaccount.com:R gs://bucket-22062018-104711/Sample-SQL-File-10-Rows.sql.gz
  ```
  * In the above example:
    ```
    * service_account_associated_with_sql_instance = adnrtynbqzayvhpovum46waf64@speckle-umbrella-38.iam.gserviceaccount.com
    * bucket_name = bucket-22062018-104711
    * file_name_with_extension = Sample-SQL-File-10-Rows.sql.gz
    ```

#### Import .gz file
* `gcloud sql` utility allows you to directly import .gz files as shown below:
  ```
  gcloud sql import sql <instance_name> gs://<bucket_name>/<file_name_with_extension> -d <database_name>
  ```
  Example:
  ```
  gcloud sql import sql sqlinstance-22062018-10572 gs://bucket-22062018-104711/Sample-SQL-File-10-Rows.sql.gz -d sample_db
  ```
  In the above example:
    * `instance_name` = `sqlinstance-22062018-10572`
    * `bucket_name` = `bucket-22062018-104711`
    * `file_name_with_extension` = `Sample-SQL-File-10-Rows.sql.gz`
    * `database_name` = `sample_db`

#### Errors
##### `ERROR: (gcloud.sql.import.sql) ERROR_RDBMS`
1. Are you trying to import a file that doesn't have correct permissions set as described in permissions section above?
2. Does any of your SQL statements have `ENGINE=MyISAM`
  Example:
  ```
  CREATE TABLE IF NOT EXISTS `user_details` (
    `user_id` int(11) NOT NULL AUTO_INCREMENT,
    `username` varchar(255) DEFAULT NULL,
    `first_name` varchar(50) DEFAULT NULL,
    `last_name` varchar(50) DEFAULT NULL,
    `gender` varchar(10) DEFAULT NULL,
    `password` varchar(50) DEFAULT NULL,
    `status` tinyint(10) DEFAULT NULL,
    PRIMARY KEY (`user_id`)
  ) ENGINE=MyISAM  DEFAULT CHARSET=latin1 AUTO_INCREMENT=1000001 ;
  ```
  
    * To resolve this:
      * Remove all occurrences of `ENGINE=MyISAM`
      * Delete the existing database
      * Re-create the database
      * Try to import again...
3. Does any of your SQL statements have multiple `INSERT INTO` statements that can be flattened out?
Example:
```
INSERT INTO `user_details` (`user_id`, `username`, `first_name`, `last_name`, `gender`, `password`, `status`) VALUES
(1, 'rogers63', 'david', 'john', 'Female', 'e6a33eee180b07e563d74fee8c2c66b8', 1),
(2, 'mike28', 'rogers', 'paul', 'Male', '2e7dc6b8a1598f4f75c3eaa47958ee2f', 1),
(3, 'rivera92', 'david', 'john', 'Male', '1c3a8e03f448d211904161a6f5849b68', 1),
(4, 'ross95', 'maria', 'sanders', 'Male', '62f0a68a4179c5cdd997189760cbcf18', 1),
(5, 'paul85', 'morris', 'miller', 'Female', '61bd060b07bddfecccea56a82b850ecf', 1);
INSERT INTO `user_details` (`user_id`, `username`, `first_name`, `last_name`, `gender`, `password`, `status`) VALUES
(6, 'smith34', 'daniel', 'michael', 'Female', '7055b3d9f5cb2829c26cd7e0e601cde5', 1),
(7, 'james84', 'sanders', 'paul', 'Female', 'b7f72d6eb92b45458020748c8d1a3573', 1),
(8, 'daniel53', 'mark', 'mike', 'Male', '299cbf7171ad1b2967408ed200b4e26c', 1),
(9, 'brooks80', 'morgan', 'maria', 'Female', 'aa736a35dc15934d67c0a999dccff8f6', 1),
(10, 'morgan65', 'paul', 'miller', 'Female', 'a28dca31f5aa5792e1cefd1dfd098569', 1);
```
  * Search for `--extended-insert` on [Cloud SQL - Diagnose Issues](https://cloud.google.com/sql/docs/mysql/diagnose-issues) page

##### `ERROR: (gcloud.sql.import.sql) INTERNAL_ERROR`
* This is usually the error reported when you are trying to import data in a database when there's already data present in the database.
* To resolve this, delete your existing database and re-create the database again
```
gcloud sql databases delete <database_name> -i <instance_name>
gcloud sql databases create <database_name> -i <instance_name>
```
* You can create the database with same name again.
