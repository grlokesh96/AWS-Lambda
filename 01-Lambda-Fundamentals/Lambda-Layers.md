# AWS Lambda Layers

## 1. Introduction

A Lambda Layer is a way to package and share libraries, dependencies, or other files across Lambda functions.

Instead of adding the same dependency separately to every function package, you can place it in a layer and attach that layer to multiple functions.

## 2. Why do we use Lambda Layers?

Imagine that five Lambda functions all use the same Python library.

Without layers, you may package the library five times.

With a layer, you package the shared dependency once and attach the layer to the functions that need it.

## 3. How layers work

```text
             Lambda Layer
          (Shared libraries)
                  |
       +----------+----------+
       |          |          |
       v          v          v
   Function A  Function B  Function C
       |          |          |
       v          v          v
    Executes   Executes   Executes
```

A function can use multiple layers, subject to AWS quotas.

## 4. Example: Create a Python layer

The following example packages a Python dependency.

### Step 1: Create the folder

```bash
mkdir -p python-layer/python
```

### Step 2: Install a dependency

```bash
pip install requests -t python-layer/python/
```

### Step 3: Create the ZIP file

```bash
cd python-layer
zip -r ../requests-layer.zip python
cd ..
```

### Step 4: Publish the layer

```bash
aws lambda publish-layer-version \
  --layer-name requests-layer \
  --zip-file fileb://requests-layer.zip \
  --compatible-runtimes python3.12
```

Use a Python version that is currently supported by Lambda and compatible with the dependency.

### Step 5: Attach the layer

First, copy the layer version ARN returned by the publish command.

```bash
aws lambda update-function-configuration \
  --function-name my-lambda-function \
  --layers <LAYER_VERSION_ARN>
```

This replaces the function's configured layer list. Include any other layers you want to retain.

## 5. Benefits

- Reuse common dependencies.
- Reduce repeated packaging work.
- Keep function code packages smaller.
- Share common libraries across teams.
- Simplify dependency maintenance.

## 6. Important points

- A layer is versioned.
- A function must be configured to use a layer version.
- The layer contents must match the runtime and architecture requirements.
- A layer does not automatically update every function when a new version is published.
- Check the current Lambda quotas for layer count and combined package size.

## 7. DevOps use case

A DevOps team maintains a shared Python layer containing approved libraries used by multiple automation functions.

This makes dependency management more consistent across projects.

## 8. Interview questions

**Q1. What is a Lambda Layer?**

A Lambda Layer is a package of shared libraries or dependencies that can be attached to Lambda functions.

**Q2. Why do we use Lambda Layers?**

To avoid packaging the same dependencies repeatedly and to simplify reuse.

**Q3. Does publishing a new layer version automatically update existing functions?**

No. Functions must be configured to use the new layer version.

## Summary

Lambda Layers help teams share and manage dependencies across multiple Lambda functions.
