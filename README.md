# wercker-beanstalk-example
Our way of using wercker to deploy to Elastic Beanstalk

## Wercker.yml explanation
### dev
It uses the same base docker as the build will use. Wercker runs in a special mode where the source directory is accessed using a symlink. This allows `npm run watch` to see file changes immediately and by changing `internal/watch` to `internal/shell` you can do stuff like `npm install package --save` and have it reflected on the package.json.

### build
Uses our node image that is just node + export 8080. Dependencies for production is installed and packed up with the source in a docker image that gets pushed before tests. We don't care about that we are creating image versions that might not be used. Last devDependencies is installed and tests are run. This creates a docker image that has different layers for node_modules and our source without including dependencies we need to run tests.

### deploy
The deploy step uses a docker image that can push to AWS. AWS doesn't support changing the tag in the docker repository in an easy way so we have to create a completely new version each time. The template file is modyfied to use the docker image tag we want and is then pushed to S3. A new Beanstalk application version is created (unless it already exists) and the environment is updated to use the new application version.

## Dockerrun.aws.json.template explanation
`Image.Name` specifies what docker image to run, including the version tag. `Image.Update` is set to false since we regard a version to be immutable. `Authentication` points to a copy of a `.dockercfg` that grants access to our private repository.
