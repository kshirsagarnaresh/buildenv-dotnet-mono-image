# DotNet 6 Build Environment Image

This project builds a Docker image that contains the tooling necessary to build .NET projects.

### Usage

#### As an SEPG Build Environment Image
The Docker image should be specified in the `official-build.props` file:
```
SEPG_BUILD_ENV_IMAGE=cafapi/buildenv-dotnet:1.0.0
```
Or for the current pre-release version:
```
SEPG_BUILD_ENV_IMAGE=cafapi/prereleases:buildenv-dotnet-1.1.0-SNAPSHOT
```

### Local Use
The image can also be used locally, for example in a WSL environment.

Create a convenience function along the following lines - you may wish to vary some details to suit your environment:
```
function buildenv-dotnet {
  local cDrive;
  local userProfile;
  local wslUserProfile="$(wslpath "$(cmd.exe /C "echo | set /p=%USERPROFILE%" 2>/dev/null)")";

  # Set variables for WSL 1 or for WSL 2
  if [[ -n "${WSL_INTEROP}" ]]; then
    cDrive='/mnt/c';
    userProfile="${wslUserProfile}";
  else
    cDrive='/run/desktop/mnt/host/c';
    userProfile=$(echo ${wslUserProfile} | sed -E 's#^/mnt/#/run/desktop/mnt/host/#');
  fi

  docker container run -it --rm \
    --network=host \
    -h buildenv-dotnet \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v "${cDrive}":/mnt/c \
    -v buildenv-dotnet_home:/root \
    -v "${userProfile}"/.gnupg:/root/.gnupg \
    -v "${userProfile}"/.m2:/root/.m2 \
    -e HTTP_PROXY -e HTTPS_PROXY -e NO_PROXY \
    -e http_proxy -e https_proxy -e no_proxy \
    -w "$(pwd)" \
    cafapi/prereleases:buildenv-dotnet-1.1.0-SNAPSHOT "$@";
}
```

You can use the function to run tools that are in the build environment image:
```
buildenv-dotnet dotnet build
```

If you find this useful then it may be convenient to add the function to your `~/.bashrc` file so that you have it readily available to you.
