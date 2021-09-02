# Supabase CLI Tour
If you are new to the Supabase CLI, or migration tools in general, it may not be easy to grasp why it's so useful. Well, what better way to introduce it than a guided tour? Now, without further ado...
## Prerequisites
This tour assumes you have the following installed in your environment:
- git
- Node.js
- Docker (make sure the daemon is up and running)
- A Postgres client (can be psql, SQL IDES, etc.)
- Supabase CLI (follow the instructions [here](https://github.com/supabase/cli/tree/new) to install)

You also need a new Supabase project. Take note of the DB connection string in `Settings > Database > Connection string`. Run the following in the SQL editor after creating the project (this is a workaround - it'll be unnecessary in the future):
```
GRANT ALL ON SEQUENCE auth.refresh_tokens_id_seq TO postgres;
```

Once you have the above set up, clone this repository and `cd` to `examples/tour`.
## 1. Initialize the project
The first thing we need to do is to initialize the Supabase CLI on the current project:
```sh
git init .    # the CLI only works in git root
supabase init
```
This will create a `supabase` directory which is managed by the CLI. Then we need to link the current project with the deploy database. Use the connection string from your Supabase project here.
```sh
supabase link --url 'postgresql://postgres:<your_password>@db.<your_project_ref>.supabase.co:5432/postgres'
```
Why do we need to do this? Because if we want to manage a database with a migration tool, we need a *baseline* where both the migration tool and the database have consistent schemas. `supabase link` does this synchronization between the two.

You'll notice that `supabase/migrations` is now populated with 2 migrations: `..._init.sql` and `..._link.sql`. `..._init.sql` is the automatically generated initial migration, which represents the schema of a newly created Supabase project. `..._link.sql` is empty because we haven't changed the schema on the Supabase project yet (which is expected!).

You'll also notice that `supabase/database` is now populated with SQL for some functions and tables. This is what we call the *structured dump*. It represents a declarative state of your database at the tip of the migrations.

Now we can start the local development server by running:
```sh
supabase start
```
Keep `supabase start` running while we do the following steps. If this is your first time running the command, this might take a while. Take note of the `API URL` and `anon key` and replace the placeholders in `index.js` with them.

With that, we can do... absolutely nothing, because we haven't set up your schema yet!
## 2. Changing schema
To change the schema for the local database, simply run some SQL against the `DB URL` from `supabase start`. For the tour, we'll run these changes using a Postgres client of your choice:
```sql
CREATE TABLE my_table(
	id int4 PRIMARY KEY,
	name text
);
```
Now we have the `my_table` table in the local database, but how do we incorporate this into migrations? If you've used other migration tools, you might be used to writing the migrations manually, but here we just need to run:
```sh
supabase db dump --name add_my_table
```
This will create a new migration named `<timestamp>_add_my_table.sql` that represents any changes we've done to the local database since `supabase start`.

It won't be interesting without sample data in the table, but instead of inserting it directly, we use the seed script in `supabase/seed.sql`. This way we don't have to run it manually every time we run `supabase start`.
```sql
-- in supabase/seed.sql
INSERT INTO my_table(id, name) VALUES (1, 'Indiana Jones'), (2, 'Marion Ravenwood');
```
Now run the following to rerun the migration scripts and the seed script:
```sql
supabase db restore
```
Then run the following:
```sh
npm clean-install
npm run start
```
You should now see the contents of `my_table`. Hooray!
## 3. Resetting schema changes
What if we ran some nasty SQL on the local database and want to wipe it all clean?
```sql
-- run on local database
ALTER TABLE my_table ADD occupation text DEFAULT 'tomb raider';
```
No need to rerun `supabase start`  here, just run:
```sh
supabase db restore
```
And the local database will be reset.
## 4. Deploying migrations
Finally, we need to deploy all these local changes to the deploy database, i.e. the Supabase project DB. Once you're happy with your schema changes, run:
```sh
supabase deploy
```
Now you can check the table editor, and you'll see the changes are now live! There's no data of course, since we only had seed data.