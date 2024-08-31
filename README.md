# multiarch-build--action

This github action is used to build multiarch oci images  using buildsafe cli

## Inputs

These are some of the inputs that are required for the action to run:

| Name                   | Description                                                                 | Required |
|------------------------|-----------------------------------------------------------------------------|----------|
| `oci_registry_username` | The username of the registry                                                | true     |
| `oci_registry_password` | The password of the registry                                                | true     |
| `ociBlocks`             | The oci block to build                                                      | true     |
| `image_name`            | The name of the image                                                       | true     |
| `tag`                   | The tag of the image                                                        | true     |
| `directory`             | The directory where the project is initialized with BSF (default value `.`) | false    |

## Example Usage

```yaml
name: Test Custom Actions

on:
  release:

jobs:
  prepare:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Run Prepare Action
        uses: buildsafedev/multiarch-build--action/prepare-action@main
        with:
          oci_registry_username: ${{ secrets.DOCKERHUB_USERNAME }}
          oci_registry_password: ${{ secrets.DOCKERHUB_PASSWORD }}
          image_name: holiodin01/testbaseimg
          tag: ${{ github.ref }}

  build:
    needs: prepare
    strategy:
      fail-fast: false
      matrix:
        # Define the matrix of platforms to build (linux/amd64, linux/arm64)....     
        platform: [ubuntu-latest, linux-arm64]  
    runs-on: ${{ matrix.platform }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Run Build Action
        uses: buildsafedev/multiarch-build--action/build-action@main
        with:
          oci_registry_username: ${{ secrets.DOCKERHUB_USERNAME }}
          oci_registry_password: ${{ secrets.DOCKERHUB_PASSWORD }}
          ociBlocks: go-dev go-runtime
          directory: '.'

  merge-godev:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Run Merge Action
        uses: buildsafedev/multiarch-build--action/merge-action@main
        with:
          oci_registry_username: ${{ secrets.DOCKERHUB_USERNAME }}
          oci_registry_password: ${{ secrets.DOCKERHUB_PASSWORD }}
          image_name: holiodin01/testbaseimg
          ociBlock: go-dev
          # using github release tag as the tag for the image          
          tag: ${{github.ref_name}}

  merge-goruntime:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Run Merge Action
        uses: buildsafedev/multiarch-build--action/merge-action@main
        with:
          oci_registry_username: ${{ secrets.DOCKERHUB_USERNAME }}
          oci_registry_password: ${{ secrets.DOCKERHUB_PASSWORD }}
          image_name: holiodin01/testbaseimg
          ociBlock: go-runtime         
          tag: ${{github.ref_name}}

```


