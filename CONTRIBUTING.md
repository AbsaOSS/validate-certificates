# How to Contribute?

## **Identifying and Reporting Bugs**
* **Ensure the bug has not already been reported** by searching our **[GitHub Issues](https://github.com/AbsaOSS/validate-certificates/issues)**.
* If you cannot find an open issue describing the problem, use the **Bug report** template to open a new one. Tag it with the **bug** label.

## **Proposing New Features**

* **Check if the feature has already been requested** by searching through our **[GitHub Issues](https://github.com/AbsaOSS/validate-certificates/issues)**.
* If the feature request doesn't exist, feel free to create a new one. Tag it with the **enhancement** label.

## **Contributing to Development**

* Check _Issues_ logs for the desired feature or bug. Ensure that no one else is already working on it.
    * If the feature/bug is not yet filed, please create a detailed issue first:
        * **"Detail Your Idea or Issue"**
* Fork the repository.
* Begin coding. Feel free to ask questions and collaborate with us.
    * Commit messages should reference the GitHub Issue and provide a concise description:
        * **"#34 Fix date parsing on macOS runners"**
    * Remember to include tests or a reproduction case for bug fixes.
* Once done, push to your fork and submit a Pull Request to our `master` branch:
    * Pull Request titles should begin with the GitHub Issue number:
        * **"45 Add support for PKCS#12 bundles"**
    * Ensure the Pull Request description clearly outlines your solution.
    * Link your PR to the relevant _Issue_.

## Branch Naming
Branches MUST start with one of the allowed prefixes: `feature/`, `fix/`, `docs/`, `chore/`

Examples:
- `feature/add-chain-validation`
- `fix/567-handle-empty-cert-array`
- `docs/improve-usage-examples`
- `chore/update-ci-runner-version`

Rename if needed before pushing:
```shell
git branch -m fix/<new-name>
```
Use lowercase kebab-case and reflect actual scope.

### Community and Communication

If you have any questions or need help, don't hesitate to reach out through our GitHub discussion section. We're here to help!

#### Thanks!

Your contributions are invaluable to us. Thank you for being part of the AbsaOSS community and helping us grow and improve!

The AbsaOSS Team
