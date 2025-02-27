# React Interview Questions and Answers

## Basic React Concepts

1. **What is React?**
   React is an open-source JavaScript library for building user interfaces, maintained by Facebook and a community of developers. It allows for the creation of reusable UI components and efficient state management.

2. **What are the key features of React?**
   - Virtual DOM for efficient rendering
   - JSX syntax
   - Component-based architecture
   - Unidirectional data flow
   - React Native for mobile app development

3. **What is JSX?**
   JSX is a syntax extension for JavaScript that looks similar to XML or HTML. It allows you to write HTML-like structures in the same file as your JavaScript code, which is then compiled to regular JavaScript objects.

4. **What is the difference between state and props?**
   - State: Internal data storage of a component that can change over time
   - Props: External data passed to a component from its parent, which are read-only

5. **What are hooks in React?**
   Hooks are functions that let you use state and other React features in functional components without writing a class. They were introduced in React 16.8.

## Advanced React Concepts

6. **Explain the useEffect hook.**
   useEffect is a hook for performing side effects in functional components. It combines the functionality of componentDidMount, componentDidUpdate, and componentWillUnmount from class components into a single API.

7. **What is the virtual DOM and how does it improve performance?**
   The virtual DOM is a lightweight copy of the actual DOM. React uses it to minimize direct DOM manipulation, improving performance by only updating the real DOM with necessary changes after comparing the current and previous virtual DOM trees.

8. **Why are keys important in React lists?**
   Keys help React identify which items in a list have changed, been added, or removed. They provide a stable identity to elements inside an array, improving the efficiency of rendering lists.

9. **What is Redux and why might it be used with React?**
   Redux is a state management library often used with React for managing global application state in a predictable way, especially useful in large-scale applications with complex state management needs.

10. **What are controlled and uncontrolled components?**
    - Controlled components: Form elements whose values are controlled by React state
    - Uncontrolled components: Form elements that maintain their own internal state

## Questions Based on Your Internship Experience

11. **How did you implement role-based routing in your project?**
    I used the JWT payload to determine the user's role and implemented conditional routing logic. Based on the role information, users were directed to their appropriate dashboards (e.g., student dashboard or mentor dashboard) upon login.

12. **Explain your process for creating reusable components with CSS modules.**
    I designed components with modularity in mind, using CSS modules to create scoped styles. This approach allowed for easy reuse across different parts of the application while maintaining style isolation and preventing conflicts.

13. **How did you handle JWT authentication on the frontend?**
    I stored the JWT in the browser's local storage upon successful login. For subsequent API requests, I included the token in the Authorization header. I also implemented token expiration checks and refresh mechanisms to maintain user sessions securely.

14. **Describe your experience with API integration in React.**
    I used the Fetch API and Axios for making HTTP requests to our backend services. This involved handling asynchronous operations, managing loading states, and error handling to ensure a smooth user experience during data fetching and submission.

15. **How did you ensure your components were responsive and user-friendly?**
    I employed responsive design techniques, using CSS flexbox and grid layouts. I also implemented media queries to adjust layouts for different screen sizes and tested extensively across various devices to ensure a consistent user experience.

16. **What challenges did you face when implementing the dashboard, and how did you overcome them?**
    One challenge was efficiently organizing and displaying large amounts of data. I solved this by implementing pagination and lazy loading techniques, and by creating modular dashboard widgets that could be easily rearranged or customized based on user preferences.

17. **How did you handle state management in your React application?**
    For local component state, I primarily used the useState hook. For more complex state management across components, I utilized the Context API to avoid prop drilling. In some cases where global state became more complex, I considered implementing Redux.

18. **Can you explain how you optimized the performance of your React components?**
    I used React.memo for functional components and PureComponent for class components to prevent unnecessary re-renders. I also implemented code splitting using React.lazy and Suspense to improve initial load times, especially for larger components like dashboards.

19. **How did you ensure the security of user data in your frontend implementation?**
    Besides using JWT for authentication, I implemented secure practices such as sanitizing user inputs, using HTTPS for all API calls, and avoiding the storage of sensitive information in local storage. I also made sure to handle errors securely without exposing sensitive details to the client.

20. **What was your approach to testing the React components you developed?**
    I wrote unit tests for individual components using Jest and React Testing Library. For more complex interactions, I created integration tests. I also performed manual testing across different browsers and devices to ensure cross-platform compatibility.
