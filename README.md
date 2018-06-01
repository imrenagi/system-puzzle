# Insight DevOps Engineering Systems Puzzle


# COMMENT

3 Major bugs found:
1. Flask server was running on its default port (5000), where it is supposed to be port 5001 because nginx and dockerfile show that they are using port 5001 instead of 5000. So, I added this modification in `app.py` so that the web server runs on port 5001.
```
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001)
```
2. Port used in nginx configuration in `docker-compose.yml` was wrong. Previously, it was "80:8080" where it is supposed to be "8080:80". The latest one means that port 8080 on the host must be forwarded to port 80 on the container. It is quite simple to find this mistake. I took a look into `conf.d/flaskapp.conf` and found that nginx is running on port 80. It means the port 80 should be opened on the container and port 8080 should be on the host side (this is also instructed on the readme :p)

3. By using those fixes, I can see the web is up and running. However, after I submit an item on the web UI, I only see an empty array `[]` returned. So, I took a look into `app.py` and made some modification so that the items can be returned in json format. The modification is shown below:
```
@app.route("/success")
def success():
    results = []
    qry = db_session.query(Items)
    results = qry.all()
    arr = []
    for item in results:
        lineItem = {}
        lineItem["name"] = item.name
        lineItem["quantity"] = item.quantity
        lineItem["description"] = item.description
        lineItem["date_added"] = item.date_added
        arr.append(lineItem)
    return jsonify(items=arr)
```

## STORIES

To start searching the bug, I start with `docker-compose.yaml` to see what services that will be spawned in order to
run this application. Then, based on the instruction on the readme, I tried to follow those steps one by one. However, instead of run `docker-compose up -d`, I tried to run the application by running `docker-compose up` so that I can see the log of the running service. Since everything looked fine, I tried to open localhost:8080 in browser, but no luck.

Then, I went back to `docker-compose.yaml` to see the order of the services startup. The very first service that must be running is postgres db, which ended up run normally. Then I go to the next service, which is the flask server. In the beginning, I tried to go to the bash shell of flask server container and make sure that the flask server runs on the correct port (which is port 5000 by default). However, I found out that the dockerfile exposed port 5001 instead and nginx also use port 5001 in its configuration. Thus, I assumed that I need to update the code on the flask server so that it runs on the port 5001 as explained on the points above. After that, I make sure that the flask server run properly on port 5001 by using `curl` from the inside of the container.

The next bug was a bit tricky since I barely remember the way we write the port configuration in `docker -p` command or in `docker-compose`. So I went to google and then suddenly I realized there was something wrong with the `docker-compose.yaml` for nginx as I explained above. So, I change it and then the web server run.

The last bug I found was the empty array returned after user submit the item. I did small work around by updating the `app.py` so that it returns json array of the items and I did it.


Thanks!
Imre <imre.nagi2812@gmail.com>

## Table of Contents
1. [Understanding the puzzle](README.md#understanding-the-puzzle)
2. [Introduction](README.md#introduction)
3. [Puzzle details](README.md#puzzle-details)
4. [Instructions to submit your solution](README.md#instructions-to-submit-your-solution)
5. [FAQ](README.md#faq)

# Understanding the puzzle

We highly recommend that you take a few dedicated minutes to read this README in its entirety before starting to think about potential solutions. You'll probably find it useful to review the codebase and understand the system at a high-level before attempting to find specific bugs.

# Introduction

Imagine you're on an engineering team that is building an eCommerce site where users can buy and sell items (similar to Etsy or eBay). One of the developers on your team has put together a very simple prototype for a system that writes and reads to a database. The developer is using Postgres for the backend database, the Python Flask framework as an application server, and nginx as a web server. All of this is developed with the Docker Engine, and put together with Docker Compose.

Unfortunately, the developer is new to many of these tools, and is having a number of issues. The developer needs your help debugging the system and getting it to work properly.

# Puzzle details

The codebase included in this repo is nearly functional, but has a few bugs that are preventing it from working properly. The goal of this puzzle is to find these bugs and fix them. To do this, you'll have to familiarize yourself with the various technologies (Docker, nginx, Flask, and Postgres) enough to figure out  You definitely don't have to be an expert on these, but you should know them well enough to understand what the problem is.

Assuming you have the Docker Engine and Docker Compose already installed, the developer said that the steps for running the system is to open a terminal, `cd` into this repo, and then enter these two commands:

    docker-compose up -d db
    docker-compose run --rm flaskapp /bin/bash -c "cd /opt/services/flaskapp/src && python -c  'import database; database.init_db()'"

This "bootstraps" the PostgreSQL database with the correct tables. After that you can run the whole system with:

    docker-compose up -d

At that point, the web application should be visible by going to `localhost:8080` in a web browser.

Once you've corrected the bugs and have the basic features working, commit the functional codebase to a new repo following the instructions below. As you debug the system, you should keep track of your thought process and what steps you took to solve the puzzle.

## Instructions to submit your solution
* Don't schedule your interview until you've worked on the puzzle
* To submit your entry please use the link you received in your systems puzzle invitation
* You will only be able to submit through the link one time
* For security, we will not open solutions submitted via files
* Use the submission box to enter the link to your GitHub repo or Bitbucket ONLY
* Link to the specific repo for this project, not your general profile
* Put any comments in the README inside your project repo

# FAQ

Here are some common questions we've received. If you have additional questions, please email us at `devops@insightdata.com` and we'll answer your questions as quickly as we can (during PST business hours), and update this FAQ. Again, only contact us after you have read through the Readme and FAQ one more time and cannot find the answer to your question.

### Which Github link should I submit?
You should submit the URL for the top-level root of your repository. For example, this repo would be submitted by copying the URL `https://github.com/InsightDataScience/systems-puzzle` into the appropriate field on the application. **Do NOT try to submit your coding puzzle using a pull request**, which would make your source code publicly available.

### Do I need a private Github repo?
No, you may use a public repo, there is no need to purchase a private repo. You may also submit a link to a Bitbucket repo if you prefer.

### What sort of system should I use to run my program (Windows, Linux, Mac)?
You should use Docker to run and test your solution, which should work on any operating system. If you're unfamiliar with Docker, we recommend attending one of our Online Tech Talks on Docker, which you should've received information about in your invitation. Alternatively, there are ample free resources available on docker.com.

### How will my solution be evaluated?
While we will review your submission briefly before your interview, the main point of this puzzle is to serve as content for discussion during the interview. In the interview, we'll evaluate your problem solving and debugging skills based off how you solved this puzzle, so be sure to document your thought process.

### This eCommerce site is ugly...should I improve the design?  
No, you should focus on the functionality. Your engineering team will bring on a designer and front-end developer later in the process, so don't worry about that aspect in this puzzle. If you have extra time, it would be far better to focus on aspects that make the code cleaner and easier to use, like tests and refactoring.

### Should I use orchestration tools like Kubernetes?
While technologies like Kubernetes are quite powerful, they're likely overkill for the simple application in this puzzle. We recommend that you stick to Docker Compose for this puzzle.
