# Dynamically Increment Application Version in Jenkins Pipeline

## Technologies Used

- Jenkins
- Docker
- GitLab
- Git
- Java
- Maven

## Project Description

1. Configured CI step: Increment patch version.
2. Configured CI step: Build Java application and clean old artifacts.
3. Configured CI step: Build Image with dynamic Docker Image tag.
4. Configured CI step: Push Image to private Docker Hub repository.
5. Configured CI step: Commit version update of Jenkins back to Git repository.
6. Configured Jenkins pipeline to not trigger automatically on CI build commit to avoid commit loop.



**I have included a .md file (markdown file) to specify the Bash CLI commands and steps used to carry out specific tasks in the project.**

## Acknowledgement

I would like to thank Nana and TWN for the opportunity to delve into the world of DevOps engineering through the TWN DevOps Bootcamp. Nana's approach to teaching is very effective and I feel concepts are introduced with the right amount of depth, whilst ensuring you are not overwhelmed with technical jargon, techniques, or tools. She has made a task that initially felt like a mountain to climb without directions, straightforward. I no longer had to worry about whether I was learning the correct concepts or missing out on any important information or skills. Thanks TWN!
