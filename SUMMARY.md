# Table of contents

* [Developer Platform API](README.md)
  * ```yaml
    type: builtin:openapi
    props:
      models: true
      downloadLink: true
    dependencies:
      spec:
        ref:
          kind: openapi
          spec: test-api
    ```

## Reference

* ```yaml
  props:
    models: false
  type: builtin:openapi
  dependencies:
    spec:
      ref:
        kind: openapi
        spec: gitbook-petstore
  ```
