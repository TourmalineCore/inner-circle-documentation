# Checklist for Submitting Pull Request for Review

Before you submit your PR for a review, you need to make sure that:

- name of your PR is informative and references the issue from the board
<br/>
e.g. *"build: #47: update vite and typescript"*
- name of your PR contains the **reason why** this change has been made (if applicable)<br/>
e.g. *"feat: add redirect from shorthand /b that is used in QR code to make links as short as possible to /books"*
- the fix or new functionality is covered by tests
- all tests pass locally and in pipeline. If they don't pass for a known reason and fixing it is out of scope, please inform your colleagues about it  in the chat
- all conflicts with the target branch are resolved
- unusual functionality has comments and link to the source of this solution (if applicable), things that can cause confusion are also accompanied by comments
- no console logs or unnecessary commented out code
- code style and naming conventions are followed
