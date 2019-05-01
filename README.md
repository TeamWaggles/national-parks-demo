# National Parks - Java Tomcat Application
This is an example Java Tomcat application packaged by [Habitat](https://habitat.sh). This example app has existed for some time, and another example can be found [here](https://github.com/habitat-sh/national-parks). The differences with this example versus previous examples are the following:

- `core/mongodb` - Previous examples had you build a version of mongodb that was already populated with data before the application 
- `mongo.toml` - This repo includes a `user.toml` file for overriding the default configuration of mongodb
- `core/haproxy` = This repo uses the `core/haproxy` package as a loadbalancer in front of National Parks
- Scaling - In both the `terrform/azure` and `terraform/aws` plans there is a `count` variable which allows you to scale out the web instances to demonstrate the concept of choreography vs orchestration in Habitat.


## Usage
In order run this repo, you must first install Habitat. You can find setup docs on the [Habitat Website](https://www.habitat.sh/docs/install-habitat/).

## Build/Test National-Parks App Locally: 

1. Clone this repo
2. `cd national-parks-demo`
3. Export environment variables to forward ports on the `export HAB_DOCKER_OPTS='-p 8000:8000 -p 8080:8080 -p 8085:8085 -p 9631:9631'`
4. `hab studio enter`
5. `build`
6. `source results/last_build.env`
7. Load `core/mongodb` package from the public depot:  
  `hab svc load core/mongodb`
8. Override the default configuration of mongodb:
   `hab config apply mongodb.default $(date +%s) mongo.toml`
9. Load the most recent build of national-parks: 
   `hab svc load $pkg_ident --bind database:mongodb.default`
10. Load `core/haproxy` from the public depot:
  `hab svc load core/haproxy --bind backend:national-parks.default`
11. Override the default configuration of HAProxy:
  `hab config apply haproxy.default $(date +%s) haproxy.toml`
12. Run `sup-log` to see the output of the supervisor

You should now be able to hit the front end of the national-parks site as follows:

- Directly - `http://localhost:8080/national-parks`  
- HAProxy - `http://localhost:8085/national-parks`

You can also view the admin console for HAProxy to see how the webserver was added dynamically to the load balancer:

http://localhost:8000/haproxy-stats

```
username: admin
password: password
```

### Build a new version of the application
There is also an `index.html` file in the root of the repo that updates the map of the National-Parks app to use red pins and colored map. This can be used to demonstrate the package promotion capabilities of Habitat. 

1. create a new feature branch - `git checkout -b update_homepage`
2. Bump the `pkg_version` in `habitat/plan.sh`
3. Overwrite `src/main/webapp/index.html` with the contents of the `red-index.html` in the root directory _NOTE: the index.html has a version number hard coded on line 38. Update that to your version number if you want it to match.
4. `hab studio enter` 
5. `build`




