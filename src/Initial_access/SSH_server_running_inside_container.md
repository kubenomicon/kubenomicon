# SSH server running inside container
### SSH Server running inside container
You're not really supposed to run ssh inside of a container but it can easily be done. This is not Kubernetes specific. Attack path here is getting creds to the SSH server and sshing in.
#### Defense
Don't run SSH servers inside of containers and if you do make sure they're locked down just like any other SSH server should be. 
Some information on [kubectl vs ssh](https://goteleport.com/blog/ssh-vs-kubectl/)

# Defending
> Pull requests needed ❤️ 