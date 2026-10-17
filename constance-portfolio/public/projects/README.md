# Project Pictures Guide

Drop your project screenshots, mockups, or diagrams into this folder (`public/projects/`).

### How to use an image for a project:
1. Save your image here, e.g. `case-management.png`
2. Open `src/data/site.js`
3. Set the `image` property on the project:
   ```javascript
   {
     title: 'Case Management System',
     category: 'Development',
     tags: ['Laravel', 'PHP', 'MySQL'],
     text: '...',
     image: '/projects/case-management.png', // <--- add path here
     link: 'https://...',                    // <--- optional live demo or repo link
   }
   ```
4. If a project does not have an image yet, leave `image: ''` and the card will display cleanly without any broken image placeholder.
