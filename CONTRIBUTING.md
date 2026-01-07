# Contributing to vDC-API Documentation

Thank you for your interest in improving the digitalSTROM vDC-API documentation!

## How to Contribute

### Reporting Issues

If you find errors, unclear explanations, or missing information:

1. **Check Existing Issues:** Look through existing issues to avoid duplicates
2. **Create an Issue:** Open a new issue with:
   - Clear description of the problem
   - Location in the documentation (file and section)
   - Suggested improvement (if applicable)
   - Reference to original PDF documentation (if relevant)

### Suggesting Improvements

We welcome suggestions for:
- Clarifying existing explanations
- Adding code examples
- Improving diagrams and visualizations
- Expanding on specific topics
- Fixing typos and grammatical errors

### Submitting Changes

1. **Fork the Repository**
   ```bash
   git clone https://github.com/KarlKiel/dsvDCDocs.git
   cd dsvDCDocs
   ```

2. **Create a Branch**
   ```bash
   git checkout -b improvement/short-description
   ```

3. **Make Your Changes**
   - Edit Markdown files in the `docs/` directory
   - Follow the existing style and structure
   - Add code examples where appropriate
   - Test all code examples

4. **Verify Your Changes**
   - Check Markdown formatting
   - Verify all links work
   - Ensure consistency with existing documentation

5. **Commit Your Changes**
   ```bash
   git add .
   git commit -m "Brief description of changes"
   ```

6. **Push and Create Pull Request**
   ```bash
   git push origin improvement/short-description
   ```
   Then create a pull request on GitHub.

## Documentation Style Guide

### Markdown Formatting

- **Headers:** Use ATX-style headers (`#`, `##`, `###`)
- **Code Blocks:** Use fenced code blocks with language specification
  ````markdown
  ```python
  # Python code here
  ```
  ````
- **Links:** Use reference-style links for cross-references within docs
- **Tables:** Use Markdown tables for structured data

### Writing Style

- **Clear and Concise:** Use simple, direct language
- **Active Voice:** Prefer active voice over passive
- **Examples:** Include practical code examples where helpful
- **Consistency:** Maintain consistency with existing documentation

### Technical Accuracy

- **Verify Against Original:** Check against original PDF documentation
- **Protocol Buffer Schema:** Ensure consistency with `genericVDC.proto`
- **API Versions:** Note version-specific features (v2, v2c, v3)
- **Test Code Examples:** All code examples should be tested

### Document Structure

Each documentation file should include:

1. **Overview Section:** Brief introduction to the topic
2. **Main Content:** Detailed explanations with subsections
3. **Examples:** Practical code examples
4. **Best Practices:** Implementation guidelines
5. **Next Steps:** Links to related documentation

## Content Guidelines

### Code Examples

- **Language:** Prefer Python for examples (accessible to most developers)
- **Complete:** Examples should be runnable or clearly pseudo-code
- **Commented:** Include comments explaining key concepts
- **Error Handling:** Show proper error handling

### Cross-References

- **Internal Links:** Link to related sections within the documentation
- **External References:** Reference original PDF files when appropriate
- **Protocol Buffer:** Reference specific messages in `genericVDC.proto`

### Diagrams and Visuals

- **ASCII Art:** Use ASCII diagrams for simple flows
- **Markdown Tables:** Use for structured comparisons
- **External Images:** If adding images, use SVG or PNG format

## Review Process

All contributions will be reviewed for:

1. **Technical Accuracy:** Consistency with official vDC-API specification
2. **Clarity:** Ease of understanding for target audience
3. **Completeness:** Adequate coverage of the topic
4. **Style:** Adherence to documentation style guide
5. **Examples:** Quality and accuracy of code examples

## Recognition

Contributors will be acknowledged in:
- Git commit history
- Pull request descriptions
- Project contributors list

## Questions?

If you have questions about contributing:

1. Check existing documentation in the `docs/` folder
2. Review original PDFs in `originalDocFiles/`
3. Open an issue for clarification
4. Contact the repository maintainers

## Code of Conduct

- Be respectful and constructive
- Focus on improving the documentation
- Provide helpful feedback
- Welcome newcomers

## License

By contributing to this documentation, you agree that your contributions will be licensed under the same license as the project.

Thank you for helping improve the vDC-API documentation!
