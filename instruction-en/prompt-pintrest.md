Pinterest Masonry Layout
----------------
(CC-BY-NC-SA 4.0 by karminski-牙医)

**Role & Goal:**
You are an experienced front-end development expert, proficient in HTML, CSS, and JavaScript. Your task is to create a visually polished, fully functional web page whose layout and style closely resemble the Pinterest page screenshot I provide. You should pay attention to code cleanliness, responsive design, and modern front-end development practices.

**Core Features & Page Structure:**

1.  **Overall layout**:
    *   Use a three-part layout: a fixed left sidebar, a top search bar, and a content display area occupying the main region.
    *   The page background color should be light gray or white, creating a clean, uncluttered look.

2.  **Left navigation sidebar (Sidebar)**:
    *   **Fixed positioning**: The sidebar should remain visible on the left side while the page scrolls.
    *   **Icon buttons**:
        *   At the top is the Pinterest logo icon (a red circle may be used as a placeholder).
        *   Below it are two main navigation buttons: "Home" and "Create". Use clear, minimalist icons to represent them (e.g., a house icon for Home, a plus icon for Create).
        *   When the mouse hovers over a button, there should be a visual feedback effect where the background turns gray.

3.  **Top search bar (Header/Search Bar)**:
    *   **Width**: The search bar should occupy most of the width at the top of the page, while keeping appropriate spacing from the left sidebar and the user area on the right.
    *   **Style**: A rounded-rectangle shape containing a search icon and placeholder text reading "Search". The background color is light gray to distinguish it from the page background.
    *   **Functionality**: This is purely visual; search functionality does not need to be implemented for now.

4.  **Top-right user area**:
    *   **Components**: Contains a message/notification icon, a profile icon, and a circular icon showing the user's avatar.
    *   **User avatar**: Use `assets/images/avatar.jpeg` as the user avatar image.
    *   **Interaction**: Clicking the circular user avatar should open a dropdown menu panel.
    *   **Dropdown panel**:
        *   The panel style should be clean, with a subtle shadow effect.
        *   The panel content should mimic the screenshot, including the currently logged-in user's info (avatar, username "ka molotov", email address), a "Convert to business" link, and action options such as "Add Pinterest account" and "Log out".
        *   The user avatar inside the dropdown panel also uses `assets/images/avatar.jpeg`.

5.  **Content area: card masonry grid (Masonry Grid)**:
    *   **Core layout**: This is the visual centerpiece of the page. Use CSS Grid or Flexbox, combined with JavaScript (if needed), to implement a dynamic multi-column masonry layout. Cards should arrange themselves automatically to fill the available space, avoiding large blank gaps at the bottom.
    *   **Card design (Card)**:
        *   Each card represents a "Pin", with an image as the main content.
        *   Cards have rounded corners. On mouse hover, the image gets a slight semi-transparent black overlay effect, and may show some action buttons (such as "Save", "Share", etc. — this part can be simplified).
        *   Space below the image can be reserved for a title or description, but this part may be simplified; the emphasis is on the image itself.
    *   **Responsive design**: The number of masonry columns should adapt to the screen width. Show 10 columns on wide screens, 5 columns on tablets, and 2 columns on phones.
    *   **Image content**:
        *   All masonry images are located in the `assets/thumb/` folder.
        *   The image filenames are `1.jpg` through `100.jpg`, 100 images in total.
        *   Use JavaScript to automatically load these 100 images (from `assets/thumb/1.jpg` to `assets/thumb/100.jpg`) and dynamically generate card elements to display them all in the masonry grid.
        *   No external image services are needed; all images are loaded locally.

**Tech Stack & Requirements:**

*   **HTML**: Use semantic HTML5 tags.
*   **CSS**:
    *   Use modern CSS techniques such as Flexbox and Grid layout.
    *   Do not depend on any large CSS framework (such as Bootstrap); hand-write the CSS for precise style control.
    *   Put the CSS inside a `<style>` tag so it can be previewed directly in the browser.
*   **JavaScript**:
    *   Use vanilla JavaScript (Vanilla JS) to implement the necessary interactive features, particularly the masonry layout calculation and the click-toggle of the top-right dropdown menu.
    *   Keep the code concise and easy to understand, and add necessary comments.

**Final Deliverable:**
Combine all the HTML, CSS, and JavaScript code into a single `.html` file so that I can copy it directly and open it in a browser to view the final result.
