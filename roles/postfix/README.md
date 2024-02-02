Role Name
=========

A brief description of the role goes here.

Postfix MySQL
------------
CREATE DATABASE postfix;

GRANT SELECT, INSERT, UPDATE, DELETE ON postfix.* TO 'postfix_admin'@'localhost' IDENTIFIED BY 'QVC8cR2zwKk5Dh4J';

GRANT SELECT, INSERT, UPDATE, DELETE ON postfix.* TO 'postfix_admin'@'localhost.localdomain' IDENTIFIED BY 'QVC8cR2zwKk5Dh4J';
FLUSH PRIVILEGES;


CREATE TABLE users ( email varchar(80) NOT NULL, password varchar(20) NOT NULL, quota BIGINT DEFAULT 3221225472, PRIMA
RY KEY (email) ) ENGINE=MyISAM;


3221225472