# Lecture 03

## Leveraging VCS for devops

VCS = Version control system
Software tool that teams uses to track changes

The main codebase for a project is called:

- Trunk
- Master
- Main

### Reasons why VCS is critical for devops

1. Avoiding dependency issues in modern containerized applications
	- Microservices have essentially become the default for development of new applications
	- dependency issues can break a project
	- dependency hell = JAR hell (in Java)
	- The linear path of building new features and fixing bugs is bound to clash with the parallel development of another feature by another team
2. Version control is tied to higher Devops performance
	- The "version control" was consistently one of the highest predictors of *performance*
	