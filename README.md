## Ruby-setup

- Create a new application
  `rails new . --database postgresql`

- Update the postgres user and password in config/database.yml
  `  username: postgres`
  `password: Gracewanjiru1573`

- Create the database
  `rails db:create`

- Create a scaffold for the database tables
  `rails g scaffold table_name`

- Create the routes for the application

- Create the FKs for the tables in the migration files
  `rails g migration AddColumnToTable table:references`

- Make sure you create the tables first before creating the FKs

- Run the migration files
  `rails db:migrate`

- Create the associations in the models
