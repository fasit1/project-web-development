# project-web-development

# Description

The Student Stack backend is written using TypeScript, mainly depending on Express for API routing,
Passport for user authorization, and mysql2 for database access. The backend exposes an API, meant
to be paired with the Student Stack client.

The Student Stack frontend is written using TypeScript, mainly depending on react and
react-simplified for building HTML, babel for TypeScript/ES translation, and axios for API calls.

This is a group assignment for DCST2002 at NTNU fall semester 2023. Contributors: Aksel Estensen,
Sarah Kalstad Johansen, Ulrik Ofstad and Wilhelm Sæhl Olsen.

We want to thank Felix Albrigtsen for hosting our project on his webserver for our presentation.

# Til sensor

I .zip-filen vi leverer i Inspera ligger `.env` og `.env.test` som er fullt funksjonelle, men denne
er ignored på gitlab. Derfor er det bare å kjøre `npm run build-prod-https` i klienten og
`npm start` i serveren for å komme igang.

## Development/production setup

1. Download or clone the project
2. Install npm dependencies on the client
   - Run `npm install` in the `./client` directory
   - Put the public build directory somewhere accessible
3. Register a Google API application
   - Follow Step 1 of
     [this guide](https://docs.dittofi.com/third-party-apis/oauth-2.0-apis/google-oauth-2.0-part-i)
   - Note your client id and api secret
4. Install npm dependencies on the server
   - Run `npm install` in the `./server` directory
5. Set up a database, and build the tables needed with the provided `./sql-setup/build_tables.sql`
   script
   - We have also supplied the script `./sql-setup/fill_prod_data.sql` to populate the database with
     mock data to test functionality in the client
   - At any point, you can clear the database of all data by running `./sql-setup/build_tables.sql`
     on your SQL server
6. Create and populate the environment file (`./server/.env`) as described below
   - Use a randomly generated session secret
   - Remember to insert your Google API details and callback URL
7. Build the client and start the server

   - Run `npm run start-http` or `npm run start-https` in the `./client` directory, depending on
     your preferences, for developer environments
   - Run `npm run build-prod-http` or `npm run build-prod-http` in the `./client` directory,
     depending on your preferences, for production environments
   - Run `npm start` in the `./server` directory
   - Make sure the `.env` file reflects these preferences

8. Open the application in your browser
   - The app should be running on `https://your-host-ip` when using the supplied `.env`
   - The URL for accessing the client should be visible in the console

### env file example

The following variables, modified for your needs, should be placed in `.env` in the `./server`
directory:

```shell
HTTPS=true
# Comment out this variable if you want to run the app on HTTP
PROD=true
# Comment out this variable if you want to run the app in a production environment
MYSQL_HOST=your.sql.server
MYSQL_USER=your_sql_username
MYSQL_PASSWORD=your_sql_password
MYSQL_DATABASE=your_sql_database
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
```

### Docker containers

#### Disclaimer:

Due to time limitations in this project, the app Docker container is dependent on also running the
database Docker container. This is a Docker networking issue that we have not resolved in time.
However, the database can be run as a container independently of the app container.

#### Building the containers:

We have supplied a simple setup for running a database in Docker. If so, run
`docker build -t studentstack-database ./sql-setup` from the project root directory. Then
`docker run --network bridge -p 3306:3306 -e MYSQL_ROOT_PASSWORD=Passord123 -e MYSQL_DATABASE=docker_db -e MYSQL_USER=docker_user -e MYSQL_PASSWORD=Passord123 --name=studentstack-database -d studentstack-database:latest`.

Consider this `.env` file for the server if you use our Docker database:

```shell
MYSQL_HOST=your_host_machine_ip_address
MYSQL_USER=docker_user
MYSQL_PASSWORD=Passord123
MYSQL_DATABASE=docker_db
```

You can edit the username, password, etc supplied in the `Dockerfile` before building, if so, these
changes must be reflected in the docker run command, and the .env file. You can also remove the
`INSERT` statements from the `docker_build.sql` script if you don't want mock data in the database.

We have also supplied a simple setup for running the app in a Docker container! If so, run
`docker build -t studentstack-app .` from the project root directory, then
`docker run --network bridge -p 443:443 --name=studentstack-app -d studentstack-app:latest`.

We have set up the Dockerfile for the app in such a way that you can comment/uncomment as you need
to expose the right ports. These edits must also be reflected in the `.env` file. However, `PROD`
and `DOCKER` must be set `=true`.

Additionally, you must set `const dockerIp = 'your_host_machine_ip_address'` (remember the 'single
quotes') at `./client/webpack.config.js:40`.

Consider this `.env` file for the server if you use our Docker app:

```shell
PROD=true
HTTPS=true
DOCKER=true

MYSQL_HOST=your_host_machine_ip_address
MYSQL_USER=docker_user
MYSQL_PASSWORD=Passord123
MYSQL_DATABASE=docker_db
```

### Using the client

Access to browsing posts and user profiles in our database is not restricted by being signed in. If
you want to interact with posts, you will have to register a user with the supplied form, use your
Google account, or, if you've filled the website with content with the supplied SQL script, you can
sign in with any of the usernames (bar the `[deleted]` user) in the Users table, and the password
'Passord123'.

You can browse questions by popularity, recently being posted, tags, search, or users (by visiting
their profile). You can post questions, answer questions, give comments to questions and answers,
and upvote/downvote questions and answers. If you are the owner of a question, you can mark an
answer as "best answer", and if you are the owner of any type of content, you can edit or delete it.

You can save your favorite tags, and browse them through the list on the left hand side. You can
save your favorite questions, and browse these from your profile via the profile button in the top
right corner. While visiting your profile, you can edit it, or delete your user.

### Tests

- Server tests must be configured in `./server/.env.test`
  - This configuration is similar to `.env`, but we highly recommend using a separate testing
    database
  - If you don't all your tables will be dropped
  - Use a different port number if testing while running the server
  - All data on the database listed in `.env.test` will be erased
- Run the tests with `npm test` in `./client` or `./server` depending on which tests you want to run

Example:

```sh
PORT=3001
MYSQL_HOST=your.sql.server
MYSQL_USER=your_sql_username
MYSQL_PASSWORD=your_sql_password
MYSQL_DATABASE=your_sql_database
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
```

### Expand the API

For all these examples, '?' indicates the variable might not be required for your specific needs

Adding to the api means adding a method to the `*Services` class in any (new or existing)
`./client/src/services/*Services.tsx` files:

```tsx
async yourMethod(input?: input_type) {
    try {
        // passing objects:
        const response = await axios.verb<expected_type>('/uri/for/api', input?)
        // OR passing strings into the URI:
        const response = await axios.verb<expected_type>('/uri/for/api/' + input)

        return response.data;
    } catch (error) {
        throw error;
    }
}
```

Then in `./server/src/routes/apiRouter.ts` add a route to the backend service:

```ts
// passing objects:
router.verb('/uri/for/api', input?, async (request, response)   => {
    const input? = request.body;
    // OR passing strings:
})
router.verb('/uri/for/api/:input', async (request, response)   => {
    const user? = request.user as User;
    const input? = Number(request.params.question_id);
    // only if the request needs to know which user is signed in:
    if (user == undefined) {
        response.status(401).send('No user for request');
        return;
    }
    const user_id = user.user_id;
    // for all: (replace the *Services with your backend service)
    try {
        const c = await *Services.yourMethod(input?, user_id?);
        c ? response.status(200).send(c) : response.status(404)send('* not found');
    } catch (error) {
        response.status(500).send(error);
    }
});
```

And finally, in `./server/src/services/*Services.ts`, add a method to the `*Services` class for the
router to call on:

```ts
yourMethod(input?, user_id?) {
    const query `YOUR SQL QUERY`;
    return new Promise<expected_type>((resolve, reject) => {
        pool.query(query, [input?, user_id?], (error, results: RowDataPacket[])=> {
            if (error) return reject(error);
            // Here you can process 'results' to suit your needs, and match 'expected_type'
            resolve(results);
        })
    })
}
```

### HTTPS

If you opt to use HTTPS there are some regards you should take.

The certificate and private key that is provided should be replaced by your own key and certificate.
We use a self signed certificate. This is done in the following way:

- Our setup is based on OpenSSL. It should be installed on your device. If not, please install it
  from the following link: https://www.openssl.org/source/

- Remove the old private.key file, go to the `/server/ssl` folder, open a terminal, and run this
  code:

  ```bash
  openssl genrsa -out private.key 2048
  ```

- To make your own self-signed certificate, remove the old `/server/ssl/certificate.crt` file, go to
  the `/server/ssl` folder, open a terminal, and run this code:

  ```bash
  openssl req -new -x509 -key private.key -out certificate.crt -days 365
  ```

### Google Authentication

If you do not supply a `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET` in your `.env`, the "Sign in
with Google" button will not render in the client, and Google functionality serverside will not run.
This is also true for running the Docker containers. If you want to use Google Authentication in a
production environment, you will need to add your host ip address (or hostname, if you are using a
DNS) to the allowed callback list in your Google Apps settings.
