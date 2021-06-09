# Devfile Samples API - Design

## Background
To support airgap scenarios, and to minimize latency retrieving projects, the devfile registry will need to support storing devfile samples and devfile starter projects. A corresponding API will also need to be provided that will allow tools to retrieve these projects from the registry, and in the case of devfile samples, the corresponding devfile and logo too (for the devfile registry viewer).

Devfile samples are mentioned in the extraDevfileEntries.yaml file in the root of the devfile registry source repository, and look like the following:
<img width="1233" alt="Screen Shot 2021-06-08 at 10 54 50 AM" src="https://user-images.githubusercontent.com/6880023/121229305-c48baf80-c85b-11eb-87ca-edf2ae7a216e.png">


Much like devfile stacks, each sample has a name, displayName, project type (among others), but rather than linking to a stack artifact, it links to a remote git repository containing a sample project.
Proposal

## Registry Build

**For each devfile sample:**

1) Git clone the devfile sample from its remote repository
2) Download the sample logo/icon
3) Retrieve the devfile from the stack, and validate the devfile
    - Validation will just consist of using the devfile library to verify it meets the minimum requirements
4) Convert the local git repository to a zip archive
5) Copy zip archive to container under samples/<sample-name>, along with the devfile and 
6) Update the references in the extraDevfileEntries.yaml file to point to the local zip archive

**For each devfile stack:**
  
1) Git clone / download each starter project
2) Store each starter project as a zip archive under the stack folder (e.g. java-wildfly/microprofile-config.zip)

## Storage - Option 1
Store devfile sample archives and devfile.yaml as OCI artifacts (like we do with devfile stacks). Logo stored directly on container, not as an OCI artifact (to avoid having to interact with the OCI registry every time the registry viewer loads icons).

Devfile starter projects must be stored as a regular file on the container, as the devfile spec does not support referencing OCI artifacts for starter projects (only Git/Zip URLs)

Benefits:
  
   - Allows us to work with the existing ORAS & OCI APIs to store and get/retrieve the samples.
  
Downsides:
  
   - The logo still has to be stored outside of the registry
   - The additional overhead for what amounts to just storing a zip archive seems a little excessive


## Storage - Option 2
Store devfile sample archives, devfile.yaml and sample logo directly on the container, not as OCI artifacts. The API to get these resources will not interact with the OCI registry API whatsoever. 

Benefits
  
   - Much simpler to implement.
   - Little overhead when retrieving the devfile samples
   - Sample, devfile.yaml, and sample icon are all stored in the same location

Downsides
  
   - Not really using the OCI registry

## API - New
  
### GET /samples/<sample-name>

Retrieves the specified devfile sample from the registry as a zip archive

### GET /samples/<sample-name>/devfile

Retrieves the devfile.yaml from the registry

### GET /samples/<sample-name>/icon

Retrieves the icon for the sample

## API - Existing
  
### GET /stacks/<stack-name>/<resource-name>

Retrieves the specified resource from the devfile stack folder on the container. E.g. `GET /stacks/java-wildfly/microprofile-config.zip` would get the microprofile-config starter 
