# D3.js Custom Linear Scale Vue Component

This project demonstrates a highly configurable linear scale visualization built with D3.js and integrated as a Vue 3 component within a Vite project. It features a custom non-linear scale distribution, various tick mark types, a dynamic indicator, a confidence range, and a simulation mode.

## Features

* **Custom Non-Linear Scale:** The scale's range from -10 to 10 is distributed across three segments with configurable percentages:
    * `[-1, 1]`
    * `[1, 5]` and `[-5, -1]`
    * `[5, 10]` and `[-10, -5]`
* **Detailed Tick Marks:** Includes major, minor, half, and even smaller 0.1 interval tick marks.
* **Dynamic Indicator:** A red triangle indicator points towards the scale, with configurable size, color, opacity, and distance from the scale.
* **Confidence Range:** An opaque box centered on the indicator, representing a confidence range. Its width (along the scale), perpendicular dimension, opacity, and color are configurable.
* **Orientation Toggle:** Switch between horizontal and vertical scale orientations.
* **Configurable Padding:** Adjust the space at the ends of the scale.
* **Smooth Transitions:** CSS transitions with configurable duration provide a dampened, visually appealing movement for dynamic elements.
* **Simulation Mode:** A built-in simulation updates the indicator value and confidence range randomly at a configurable frequency (1-10 Hz).
* **Responsive Design:** The scale component adapts to the size of its container, ensuring responsiveness.
* **Vue 3 Component:** Encapsulated as a reusable Vue component.
* **Vite & Tailwind CSS v4:** Modern development setup for fast development and sleek styling.

## Project Structure

````

my-d3-scale-app/
├── public/
│   └── vite.svg
├── src/
│   ├── components/
│   │   └── LinearScale.vue  \<-- The D3.js scale Vue component
│   ├── App.vue              \<-- Main application component with controls
│   ├── main.js              \<-- Vue app entry point
│   └── style.css            \<-- Tailwind CSS import and global styles
├── index.html               \<-- Main HTML file
├── package.json
├── postcss.config.js        \<-- PostCSS configuration for Tailwind
├── README.md                \<-- This file
├── tailwind.config.js       \<-- Tailwind CSS configuration
└── vite.config.js           \<-- Vite configuration

````

## Setup and Installation

To get this project up and running on your local machine, follow these steps:

1.  **Clone the repository (if applicable) or navigate to your project directory:**
    ```bash
    # If you created the project using `npm create vite@latest`
    cd my-d3-scale-app
    ```

2.  **Install project dependencies:**
    ```bash
    npm install
    ```

3.  **Install Tailwind CSS and its peer dependencies:**
    ```bash
    npm install -D tailwindcss postcss autoprefixer
    ```

4.  **Initialize Tailwind CSS config files:**
    ```bash
    npx tailwindcss init -p
    ```
    * If you encounter an `npm error: could not determine executable to run`, this is typically a Node.js/npm environment issue. Try `npm cache clean --force`, `npm install -g npm@latest`, or reinstalling Node.js.

5.  **Verify/Update Configuration Files:**
    Ensure your `tailwind.config.js`, `postcss.config.js`, `src/style.css`, and `src/main.js` files match the content provided in the Canvas.
    * **Crucially for Tailwind CSS v4:** Make sure `src/style.css` **DOES NOT** contain `@tailwind` directives.

## Running the Application

Once the setup is complete, you can start the development server:

<!-- ```bash
npm run dev
```` -->

This will typically open the application in your browser at `http://localhost:5173/`.

## Usage

The application presents the custom linear scale and a set of control sliders and inputs.

* **Adjust Indicator Value:** Use the main slider to manually set the indicator's position.
* **Indicator Settings:**
  * **Size (px):** Control the size of the triangle.
  * **Color:** Change the indicator's color.
  * **Opacity (0-1):** Adjust the indicator's transparency.
  * **Distance (% of SVG):** Set how far the indicator is from the scale line.
* **Confidence Box Settings:**
  * **Width (% of scale):** Control the length of the confidence box along the scale.
  * **Cross Dimension (px):** Adjust the width/height of the confidence box perpendicular to the scale.
  * **Opacity (0-1):** Adjust the confidence box's transparency.
  * **Color:** Change the confidence box's color.
* **Transition Duration (seconds):** Modify the smoothness of animations.
* **Orientation:** Toggle between horizontal and vertical scale layouts.
* **Scale Padding (pixels):** Add space at the ends of the scale.
* **Range Distribution Percentages:** Configure the proportional sizes of the scale's segments (`[-1, 1]`, `[-5,-1] & [1,5]`, `[-10,-5] & [5,10]`). Ensure they sum to 100%.
* **Update Scale Button:** Apply changes made in the percentage, padding, and orientation settings.
* **Simulation Controls:**
  * **Simulation Frequency (Hz):** Adjust how fast the simulated values update.
  * **Start/Stop Simulation Button:** Toggle the automatic simulation of indicator values and confidence ranges. When active, manual controls are disabled.

## Deployment to GitHub Pages

To deploy your Vite project to GitHub Pages, you need to configure Vite correctly and then build and push your project.

### 1\. Configure Vite for GitHub Pages

Edit your `vite.config.js` file (located in your project's root directory). You need to set the `base` option to your repository name.

**Example `vite.config.js`:**

````javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// [https://vitejs.dev/config/](https://vitejs.dev/config/)
export default defineConfig({
  plugins: [vue()],
  base: '/my-d3-scale-app/', // Replace 'my-d3-scale-app' with your GitHub repository name
})
````

### 2\. Build the Project

Run the build command in your terminal. This will create a `dist` folder in your project root, containing the optimized static assets for your application.

````bash
npm run build
````

### 3\. Deploy to GitHub Pages

There are two common methods for deploying the `dist` folder to GitHub Pages:

#### Method A: Manual Deployment (using `gh-pages` branch)

This method involves creating a `gh-pages` branch in your repository and pushing the contents of your `dist` folder to it.

1. **Install `gh-pages` package:**

    ````bash
    npm install gh-pages --save-dev
    ````

2. **Add a deploy script to `package.json`:**
    Open your `package.json` file and add a new script under `"scripts"`:

    ````json
    {
      "name": "my-d3-scale-app",
      "private": true,
      "version": "0.0.0",
      "type": "module",
      "scripts": {
        "dev": "vite",
        "build": "vite build",
        "preview": "vite preview",
        "deploy": "gh-pages -d dist"  <-- Add this line
      },
      "dependencies": {
        "d3": "^7.9.0",
        "vue": "^3.4.21"
      },
      "devDependencies": {
        "@vitejs/plugin-vue": "^5.0.4",
        "autoprefixer": "^10.4.19",
        "gh-pages": "^6.1.1",  <-- Make sure this is here after installation
        "postcss": "^8.4.38",
        "tailwindcss": "^4.0.0-next-11",
        "vite": "^5.2.0"
      }
    }
    ````

3. **Run the deploy script:**

    ````bash
    npm run deploy
    ````

    This command will:

      * Create a `gh-pages` branch (if it doesn't exist).
      * Commit the contents of your `dist` folder to this branch.
      * Push the `gh-pages` branch to your GitHub repository.

4. **Configure GitHub Pages in Repository Settings:**

      * Go to your GitHub repository on `github.com`.
      * Navigate to **Settings** \> **Pages**.
      * Under "Build and deployment", select **Deploy from a branch**.
      * Choose `gh-pages` as the branch and `/ (root)` as the folder.
      * Click **Save**.
      * Your site should now be live at `https://<your-github-username>.github.io/<your-repository-name>/`. It might take a few minutes for GitHub Pages to build and publish.

#### Method B: Automated Deployment with GitHub Actions (Recommended)

For continuous deployment, GitHub Actions is a more robust solution. This workflow will automatically build and deploy your project whenever you push changes to your `main` (or `master`) branch.

1. **Create a workflow file:**
    In your project's root, create the directory `.github/workflows/` and inside it, create a file named `deploy.yml`.

    ````
    .github/
    └── workflows/
        └── deploy.yml
    ````

2. **Add the workflow content to `deploy.yml`:**

    ````yaml
    name: Deploy to GitHub Pages

    on:
      push:
        branches:
          - main # or 'master' if that's your default branch

    jobs:
      build-and-deploy:
        runs-on: ubuntu-latest

        steps:
          - name: Checkout 🛎️
            uses: actions/checkout@v4

          - name: Setup Node.js
            uses: actions/setup-node@v4
            with:
              node-version: '20' # Use a recent LTS Node.js version

          - name: Install dependencies 📦
            run: npm install

          - name: Build project 🏗️
            run: npm run build

          - name: Deploy to GitHub Pages 🚀
            uses: peaceiris/actions-gh-pages@v4
            with:
              github_token: ${{ secrets.GITHUB_TOKEN }}
              publish_dir: ./dist
              # If your repository is not at the root of your GitHub Pages site,
              # you might need to set the destination branch.
              # For project pages (https://<username>.github.io/<repo-name>/),
              # the default branch is usually 'gh-pages'.
              # For user/organization pages (https://<username>.github.io/),
              # the default branch is usually 'main' or 'master'.
              publish_branch: gh-pages # This is the branch where your built site will be pushed
    ````

3. **Commit and Push:**
    Commit these changes and push them to your `main` (or `master`) branch.

    ````bash
    git add .github/workflows/deploy.yml
    git commit -m "feat: Add GitHub Pages deployment workflow"
    git push origin main # or master
    ````

4. **Configure GitHub Pages in Repository Settings:**

      * Go to your GitHub repository on `github.com`.
      * Navigate to **Settings** \> **Pages**.
      * Under "Build and deployment", select **GitHub Actions**.
      * GitHub will automatically detect the workflow.
      * Your site should now be live at `https://<your-github-username>.github.io/<your-repository-name>/` after the workflow completes. You can monitor the workflow's progress in the "Actions" tab of your repository.

<!-- end list -->
