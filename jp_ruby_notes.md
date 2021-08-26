# JP Ruby Notes








## General Notes
Double quotes enclosing a string will evaluate any variable within the string which is inside curly brackets `{}` and preceded by a pound sign `#`.  
    ```ruby
    "hello #{name}, how are you?"       ## this evals correctly
    'hello #{name}, how are you?'       ## this prints the string literal '#{name}'
    ```
