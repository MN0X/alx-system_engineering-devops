
## Postmortem

At approximately 08:45 GMT+1 (Moroccan time), following the deployment of a containerized Node.js application for ALX’s Backend Capstone project, an outage occurred on the production environment. GET requests to the API endpoint `/api/recipes` were met with a 502 Bad Gateway error, contrary to the expected JSON response containing recipe data.

## Debugging Process

The bug was brought to the attention of lead debugger SlimDev (that's me, the name’s made up but the pain was real) at 11:02 AM after reports came flooding in—by “flooding,” I mean one person pinged me on Slack. Close enough.

1. First, checked if the Node.js app was even alive. Ran `docker ps` — confirmed the container was up. So far, so good.

2. Inspected the logs inside the container using `docker logs [container_id]`. Output showed repeated connection refusals to MongoDB. Not good.

3. Verified MongoDB was running using `docker ps`. Uh-oh — no container named `mongo` was found. Mystery deepens.

4. Realized the `docker-compose.yml` file was not configured to spin up the database container. Classic case of “it worked on my machine.”

5. Started the MongoDB service manually with `docker-compose up -d mongo`. Checked logs again. Now the Node app was screaming about authentication failures. Of course.

6. Looked at the `config.js` file and compared it to the `docker-compose` secrets. Found the culprit: the environment variable `MONGO_URI` had a typo — written as `MONG_URI`. That one missing ‘O’ cost me 45 minutes.

7. Fixed the environment variable, rebuilt the containers with `docker-compose down && docker-compose up -d --build`.

8. Curled the endpoint one more time. Boom — JSON came back like a champ. Crisis averted.

## Summation

This was a two-parter:  
- Missing database container.  
- Typo in an environment variable.

Both classic rookie mistakes, both easily preventable, and both now permanently seared into my memory.

## Prevention

Here’s how to avoid this mini-drama in the future:

- **Automated Testing**: Write integration tests that hit endpoints and verify database connectivity before deployment.
- **Linter + Env Checker**: Use tools like `dotenv-linter` or a custom script to verify environment variables before startup.
- **Monitoring**: Add basic logging and status healthchecks to the app and monitor via tools like PM2 or UptimeRobot.
- **Team Rituals**: Mandate a dry-run of `docker-compose up` before pushing to production.

Oh, and always double-check your variables. Because MONG_URI isn’t just a typo—it’s a vibe killer.  
