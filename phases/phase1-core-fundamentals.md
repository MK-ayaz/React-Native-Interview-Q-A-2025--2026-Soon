<style>
  .handbook-page {
    background-color: #ffffff;
    color: #1e293b;
    line-height: 1.8;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    max-width: 900px;
    margin: 0 auto;
    padding: 20px;
  }
  .phase-header {
    background: linear-gradient(135deg, #059669 0%, #047857 100%);
    padding: 40px;
    border-radius: 24px;
    color: white;
    margin-bottom: 40px;
    text-align: center;
    box-shadow: 0 10px 25px -5px rgba(5, 150, 105, 0.2);
  }
  .phase-number {
    font-weight: 800;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    font-size: 0.875rem;
    opacity: 0.9;
    display: block;
    margin-bottom: 8px;
  }
  .phase-title {
    font-size: 2.5rem;
    font-weight: 800;
    margin: 0;
    line-height: 1.2;
  }
  .qa-card {
    background: #f8fafc;
    border: 1px solid #e2e8f0;
    border-radius: 16px;
    padding: 32px;
    margin-bottom: 32px;
    transition: transform 0.2s ease;
  }
  .question-tag {
    background: #dbeafe;
    color: #1e40af;
    padding: 4px 12px;
    border-radius: 9999px;
    font-size: 0.75rem;
    font-weight: 700;
    display: inline-block;
    margin-bottom: 16px;
  }
  .beginner-tip {
    background: #f0fdf4;
    border-left: 4px solid #22c55e;
    padding: 20px;
    border-radius: 8px;
    margin: 24px 0;
  }
  .follow-up-trap {
    background: #fef2f2;
    border-left: 4px solid #ef4444;
    padding: 20px;
    border-radius: 8px;
    margin: 24px 0;
  }
  .code-example {
    background: #1e293b;
    color: #94a3b8;
    padding: 8px 16px;
    border-top-left-radius: 8px;
    border-top-right-radius: 8px;
    font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace;
    font-size: 0.75rem;
  }
  .code-block {
    margin-top: 0;
    border-top-left-radius: 0;
    border-top-right-radius: 0;
  }
  .component-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 16px;
    margin: 24px 0;
  }
  .component-card {
    background: #ffffff;
    border: 1px solid #e2e8f0;
    padding: 16px;
    border-radius: 12px;
    text-align: center;
  }
  .nav-footer {
    display: flex;
    justify-content: space-between;
    margin-top: 60px;
    padding-top: 20px;
    border-top: 1px solid #e2e8f0;
  }
</style>

<div class="handbook-page">

<div class="phase-header">
  <span class="phase-number">Phase 01</span>
  <h1 class="phase-title">React Native Fundamentals</h1>
  <p style="margin-top: 16px; opacity: 0.9; font-size: 1.1rem;">Building Your Foundation: Core Concepts, Components, and State Management</p>
</div>

---

### 📋 Phase Overview
Before diving into advanced architecture or complex patterns, master the fundamentals that every React Native developer must know. This phase covers the essential building blocks from "What is React Native?" to basic component patterns, ensuring you can confidently discuss the framework's core concepts in any interview.

---

<div class="qa-card">
  <span class="question-tag">Q1: WHAT IS MOBILE DEVELOPMENT?</span>
  <h3 style="margin-top: 0;">Can you explain what mobile development is and why it's important?</h3>

  <p><strong>Mobile development</strong> is the process of creating software applications that run on mobile devices like smartphones and tablets. It's important because mobile devices have become the primary way people access the internet and use digital services worldwide.</p>

  <div class="component-grid">
    <div class="component-card">
      <span style="font-size: 1.5rem; font-weight: 800; color: #059669;">📱</span>
      <span style="display: block; font-size: 0.875rem; color: #64748b; text-transform: uppercase; letter-spacing: 0.05em;">Mobile-First Era</span>
    </div>
    <div class="component-card">
      <span style="font-size: 1.5rem; font-weight: 800; color: #059669;">🌐</span>
      <span style="display: block; font-size: 0.875rem; color: #64748b; text-transform: uppercase; letter-spacing: 0.05em;">Global Reach</span>
    </div>
    <div class="component-card">
      <span style="font-size: 1.5rem; font-weight: 800; color: #059669;">💡</span>
      <span style="display: block; font-size: 0.875rem; color: #64748b; text-transform: uppercase; letter-spacing: 0.05em;">Innovation</span>
    </div>
  </div>

  <div class="beginner-tip">
    <strong>🟢 Absolute Beginner Tip:</strong> Mobile apps are different from websites because they need to work offline, respond to touch gestures, and integrate with device features like camera and GPS.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">Q2: NATIVE VS HYBRID VS WEB APPS</span>
  <h3 style="margin-top: 0;">What's the difference between native, hybrid, and web apps?</h3>

  <p>Understanding the different types of mobile apps is fundamental to choosing the right development approach:</p>

  <table style="width: 100%; border-collapse: collapse; margin: 24px 0;">
    <tr style="background: #f1f5f9;">
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Type</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Technology</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Performance</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Development</th>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><strong>Native</strong></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Swift/Java/Kotlin</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Best performance</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Separate iOS/Android code</td>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><strong>Hybrid</strong></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">HTML/CSS/JS + WebView</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Good performance</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Single codebase</td>
    </tr>
    <tr>
      <td style="padding: 12px;"><strong>Web</strong></td>
      <td style="padding: 12px;">HTML/CSS/JS</td>
      <td style="padding: 12px;">Limited mobile features</td>
      <td style="padding: 12px;">Responsive websites</td>
    </tr>
  </table>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> React Native creates truly native apps (like the first row), not hybrid apps (like the second row). Your React Native code becomes real native iOS/Android components.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">Q3: WHAT IS REACT NATIVE?</span>
  <h3 style="margin-top: 0;">Explain what React Native is and how it solves mobile development challenges.</h3>

  <p><strong>React Native</strong> is an open-source framework developed by Facebook (now Meta) that allows developers to build native mobile applications using JavaScript and React. It solves the challenge of maintaining separate codebases for iOS and Android by enabling <strong>cross-platform development</strong> with a single JavaScript codebase.</p>

  <div class="component-grid">
    <div class="component-card">
      <span style="font-size: 1.5rem; font-weight: 800; color: #059669;">📱</span>
      <span style="display: block; font-size: 0.875rem; color: #64748b; text-transform: uppercase; letter-spacing: 0.05em;">Single Codebase</span>
    </div>
    <div class="component-card">
      <span style="font-size: 1.5rem; font-weight: 800; color: #059669;">⚡</span>
      <span style="display: block; font-size: 0.875rem; color: #64748b; text-transform: uppercase; letter-spacing: 0.05em;">Native Performance</span>
    </div>
    <div class="component-card">
      <span style="font-size: 1.5rem; font-weight: 800; color: #059669;">🔄</span>
      <span style="display: block; font-size: 0.875rem; color: #64748b; text-transform: uppercase; letter-spacing: 0.05em;">Hot Reload</span>
    </div>
  </div>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> React Native compiles to truly native apps, not web apps wrapped in native containers. Your JavaScript code runs on a JavaScript thread, but the UI is rendered using actual native components.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">Q4: JAVASCRIPT BASICS FOR RN</span>
  <h3 style="margin-top: 0;">What JavaScript concepts should you know before learning React Native?</h3>

  <p>React Native is built on JavaScript, so you need to understand these core concepts:</p>

  <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 16px; margin: 24px 0;">
    <div style="background: #eff6ff; border: 1px solid #dbeafe; padding: 16px; border-radius: 8px; text-align: center;">
      <strong>Variables & Data Types</strong>
      <p style="margin: 8px 0 0 0; font-size: 0.875rem; color: #64748b;">var, let, const, strings, numbers, booleans</p>
    </div>
    <div style="background: #eff6ff; border: 1px solid #dbeafe; padding: 16px; border-radius: 8px; text-align: center;">
      <strong>Functions</strong>
      <p style="margin: 8px 0 0 0; font-size: 0.875rem; color: #64748b;">Arrow functions, regular functions, callbacks</p>
    </div>
    <div style="background: #eff6ff; border: 1px solid #dbeafe; padding: 16px; border-radius: 8px; text-align: center;">
      <strong>Arrays & Objects</strong>
      <p style="margin: 8px 0 0 0; font-size: 0.875rem; color: #64748b;">Array methods, object destructuring</p>
    </div>
    <div style="background: #eff6ff; border: 1px solid #dbeafe; padding: 16px; border-radius: 8px; text-align: center;">
      <strong>ES6+ Features</strong>
      <p style="margin: 8px 0 0 0; font-size: 0.875rem; color: #64748b;">Promises, async/await, spread operator</p>
    </div>
  </div>

  <div class="code-example">Basic JavaScript Concepts</div>
  <pre class="code-block"><code class="language-javascript">// Variables
const name = "React Native";
let count = 0;

// Arrow function
const greet = (user) => `Hello, ${user}!`;

// Async function (important for RN)
const fetchData = async () => {
  try {
    const response = await fetch('https://api.example.com/data');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Error:', error);
  }
};</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> React Native uses modern JavaScript (ES6+), so features like arrow functions, destructuring, and async/await are essential. If you're new to JavaScript, focus on these before diving into React Native.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">Q5: REACT VS REACT NATIVE</span>
  <h3 style="margin-top: 0;">What's the difference between React and React Native?</h3>

  <p>While both use React's component-based architecture and JSX syntax, they target completely different platforms:</p>

  <table style="width: 100%; border-collapse: collapse; margin: 24px 0;">
    <tr style="background: #f1f5f9;">
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Aspect</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">React</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">React Native</th>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><strong>Platform</strong></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Web browsers</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Mobile (iOS/Android)</td>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><strong>Components</strong></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><code>&lt;div&gt;</code>, <code>&lt;span&gt;</code></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><code>&lt;View&gt;</code>, <code>&lt;Text&gt;</code></td>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><strong>Styling</strong></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">CSS</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">StyleSheet API</td>
    </tr>
    <tr>
      <td style="padding: 12px;"><strong>Output</strong></td>
      <td style="padding: 12px;">HTML/CSS/JS</td>
      <td style="padding: 12px;">Native mobile apps</td>
    </tr>
  </table>

  <div class="follow-up-trap">
    <strong>🪤 Follow-up Trap:</strong> Don't say "React Native is just React for mobile" - they're different frameworks that happen to share similar syntax and patterns. React Native doesn't use HTML or CSS.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">Q6: WHAT IS A COMPONENT?</span>
  <h3 style="margin-top: 0;">Explain what components are in React Native and why they're important.</h3>

  <p><strong>Components</strong> are the building blocks of React Native applications. They are reusable, self-contained pieces of code that define how a portion of your UI should look and behave. Think of them as custom HTML tags that you create yourself.</p>

  <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin: 24px 0;">
    <div style="background: #f0fdf4; border: 1px solid #bbf7d0; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #166534;">🔧 Functional Component</h4>
      <p style="margin: 0; font-size: 0.875rem; color: #374151;">Simple functions that return JSX. Most common today.</p>
    </div>
    <div style="background: #fef3c7; border: 1px solid #fde68a; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #92400e;">🏗️ Class Component</h4>
      <p style="margin: 0; font-size: 0.875rem; color: #374151;">ES6 classes with more features. Less common now.</p>
    </div>
  </div>

  <div class="code-example">Simple Functional Component</div>
  <pre class="code-block"><code class="language-javascript">import React from 'react';
import { View, Text } from 'react-native';

function WelcomeMessage() {
  return (
    &lt;View&gt;
      &lt;Text&gt;Welcome to React Native!&lt;/Text&gt;
    &lt;/View&gt;
  );
}

// Using the component
function App() {
  return (
    &lt;View&gt;
      &lt;WelcomeMessage /&gt;
      &lt;WelcomeMessage /&gt; {/* Reusable! */}
    &lt;/View&gt;
  );
}</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Components should be small, focused, and reusable. A good rule is: if you copy-paste JSX more than once, make it a component.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">Q7: JSX IN REACT NATIVE</span>
  <h3 style="margin-top: 0;">What is JSX and why is it important in React Native?</h3>

  <p><strong>JSX (JavaScript XML)</strong> is a syntax extension for JavaScript that allows you to write HTML-like code within JavaScript. It's not required but makes React Native code much more readable and maintainable.</p>

  <div class="code-example">JSX vs Plain JavaScript</div>
  <pre class="code-block"><code class="language-javascript">// Without JSX (hard to read!)
React.createElement(View, {style: styles.container},
  React.createElement(Text, {style: styles.title}, 'Hello World!'),
  React.createElement(TextInput, {
    placeholder: 'Type here...',
    onChangeText: setText
  })
);

// With JSX (much cleaner!)
&lt;View style={styles.container}&gt;
  &lt;Text style={styles.title}&gt;Hello World!&lt;/Text&gt;
  &lt;TextInput
    placeholder="Type here..."
    onChangeText={setText}
  /&gt;
&lt;/View&gt;</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> JSX gets transpiled to <code>React.createElement()</code> calls by Babel. You can use any valid JavaScript expressions inside JSX by wrapping them in curly braces: <code>{myVariable}</code>, <code>{2 + 2}</code>, or even <code>{items.map(item => &lt;Text&gt;{item}&lt;/Text&gt;)}</code>.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">Q4: CORE COMPONENTS</span>
  <h3 style="margin-top: 0;">What are the core components in React Native?</h3>

  <p>React Native provides essential components that map to native mobile UI elements:</p>

  <div class="component-grid">
    <div class="component-card">
      <span style="font-size: 1.5rem; font-weight: 800; color: #3b82f6;">📦</span>
      <span style="display: block; font-weight: 600; margin: 8px 0;">View</span>
      <span style="display: block; font-size: 0.875rem; color: #64748b;">Container (like div)</span>
    </div>
    <div class="component-card">
      <span style="font-size: 1.5rem; font-weight: 800; color: #3b82f6;">📝</span>
      <span style="display: block; font-weight: 600; margin: 8px 0;">Text</span>
      <span style="display: block; font-size: 0.875rem; color: #64748b;">Display text content</span>
    </div>
    <div class="component-card">
      <span style="font-size: 1.5rem; font-weight: 800; color: #3b82f6;">🖼️</span>
      <span style="display: block; font-weight: 600; margin: 8px 0;">Image</span>
      <span style="display: block; font-size: 0.875rem; color: #64748b;">Display images</span>
    </div>
    <div class="component-card">
      <span style="font-size: 1.5rem; font-weight: 800; color: #3b82f6;">📝</span>
      <span style="display: block; font-weight: 600; margin: 8px 0;">TextInput</span>
      <span style="display: block; font-size: 0.875rem; color: #64748b;">User text input</span>
    </div>
  </div>

  <div class="code-example">Basic Component Usage</div>
  <pre class="code-block"><code class="language-javascript">import { View, Text, Image, TextInput } from 'react-native';

export default function App() {
  return (
    &lt;View style={{flex: 1, padding: 20}}&gt;
      &lt;Text style={{fontSize: 24}}&gt;Hello React Native!&lt;/Text&gt;
      &lt;Image
        source={{uri: 'https://example.com/image.jpg'}}
        style={{width: 100, height: 100}}
      /&gt;
      &lt;TextInput
        placeholder="Type something..."
        style={{borderWidth: 1, padding: 10, marginTop: 20}}
      /&gt;
    &lt;/View&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">Q5: PROPS VS STATE</span>
  <h3 style="margin-top: 0;">Explain the difference between props and state in React Native.</h3>

  <p><strong>Props</strong> and <strong>state</strong> are both plain JavaScript objects that hold data, but they serve different purposes:</p>

  <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 24px; margin: 24px 0;">
    <div style="background: #eff6ff; border: 1px solid #dbeafe; padding: 20px; border-radius: 12px;">
      <h4 style="margin: 0 0 12px 0; color: #1e40af;">📥 Props (Properties)</h4>
      <ul style="margin: 0; padding-left: 20px;">
        <li>Passed down from parent components</li>
        <li>Immutable (read-only)</li>
        <li>Used for component configuration</li>
        <li>Trigger re-renders when changed</li>
      </ul>
    </div>
    <div style="background: #fef3c7; border: 1px solid #fde68a; padding: 20px; border-radius: 12px;">
      <h4 style="margin: 0 0 12px 0; color: #92400e;">🔄 State</h4>
      <ul style="margin: 0; padding-left: 20px;">
        <li>Managed within the component</li>
        <li>Mutable (can be changed)</li>
        <li>Used for interactive data</li>
        <li>Triggers re-renders when updated</li>
      </ul>
    </div>
  </div>

  <div class="code-example">Props vs State Example</div>
  <pre class="code-block"><code class="language-javascript">// Props: passed from parent
function Greeting({name}) { // 'name' is a prop
  return &lt;Text&gt;Hello, {name}!&lt;/Text&gt;;
}

// State: managed internally
function Counter() {
  const [count, setCount] = useState(0); // 'count' is state

  return (
    &lt;View&gt;
      &lt;Text&gt;Count: {count}&lt;/Text&gt;
      &lt;TouchableOpacity onPress={() =&gt; setCount(count + 1)}&gt;
        &lt;Text&gt;Increment&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/View&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">Q6: USESTATE HOOK</span>
  <h3 style="margin-top: 0;">How does the useState hook work in React Native?</h3>

  <p><strong>useState</strong> is a React Hook that allows functional components to have state. It returns an array with two elements: the current state value and a function to update it.</p>

  <div class="code-example">useState Basic Usage</div>
  <pre class="code-block"><code class="language-javascript">import { useState } from 'react';
import { View, Text, TouchableOpacity } from 'react-native';

function Counter() {
  // Declare state variable 'count' with initial value 0
  const [count, setCount] = useState(0);

  return (
    &lt;View style={{padding: 20}}&gt;
      &lt;Text style={{fontSize: 24}}&gt;Count: {count}&lt;/Text&gt;
      &lt;TouchableOpacity
        style={{backgroundColor: '#007AFF', padding: 10, borderRadius: 5, marginTop: 10}}
        onPress={() =&gt; setCount(count + 1)} // Update state
      &gt;
        &lt;Text style={{color: 'white'}}&gt;Increment&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/View&gt;
  );
}</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> The setter function (<code>setCount</code>) can accept a new value directly or a function that receives the previous state: <code>setCount(prevCount =&gt; prevCount + 1)</code>. Use the function form when the new state depends on the previous state.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">Q7: USEEFFECT HOOK</span>
  <h3 style="margin-top: 0;">Explain useEffect and when to use it in React Native.</h3>

  <p><strong>useEffect</strong> allows you to perform side effects in functional components. It's equivalent to lifecycle methods like <code>componentDidMount</code>, <code>componentDidUpdate</code>, and <code>componentWillUnmount</code> combined.</p>

  <div class="code-example">Common useEffect Patterns</div>
  <pre class="code-block"><code class="language-javascript">// 1. Run on every render (rarely needed)
useEffect(() =&gt; {
  console.log('Component rendered');
});

// 2. Run once on mount (empty dependency array)
useEffect(() =&gt; {
  fetchUserData(); // API call
}, []); // Empty array = run once

// 3. Run when specific values change
useEffect(() =&gt; {
  if (userId) {
    fetchUserProfile(userId);
  }
}, [userId]); // Run when userId changes

// 4. Cleanup function
useEffect(() =&gt; {
  const subscription = subscribeToUpdates();
  return () =&gt; subscription.unsubscribe(); // Cleanup
}, []);</code></pre>

  <div class="follow-up-trap">
    <strong>🪤 Follow-up Trap:</strong> Forgetting the dependency array can cause infinite re-renders or stale closures. Always include dependencies or use ESLint rules to catch this.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">Q8: COMPONENT LIFECYCLE</span>
  <h3 style="margin-top: 0;">How do component lifecycle methods work in functional components?</h3>

  <p>Functional components use Hooks instead of lifecycle methods. The main lifecycle events are handled by <code>useEffect</code>:</p>

  <table style="width: 100%; border-collapse: collapse; margin: 24px 0;">
    <tr style="background: #f1f5f9;">
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Class Component</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Functional Component</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">When it runs</th>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><code>componentDidMount</code></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><code>useEffect(() =&gt; {...}, [])</code></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">After first render</td>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><code>componentDidUpdate</code></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><code>useEffect(() =&gt; {...}, [deps])</code></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">After re-renders when deps change</td>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><code>componentWillUnmount</code></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><code>useEffect(() =&gt; { return cleanup }, [])</code></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Before component unmounts</td>
    </tr>
  </table>
</div>

<div class="qa-card">
  <span class="question-tag">Q9: STYLING IN REACT NATIVE</span>
  <h3 style="margin-top: 0;">How does styling work in React Native?</h3>

  <p>React Native uses JavaScript objects for styling, not CSS. Styles are defined using the <code>StyleSheet</code> API for performance optimization.</p>

  <div class="code-example">StyleSheet Usage</div>
  <pre class="code-block"><code class="language-javascript">import { StyleSheet, View, Text } from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 10,
    borderRadius: 5,
    marginTop: 20,
  },
});

function App() {
  return (
    &lt;View style={styles.container}&gt;
      &lt;Text style={styles.title}&gt;Hello World&lt;/Text&gt;
      &lt;View style={styles.button}&gt;
        &lt;Text style={{color: 'white'}}&gt;Press Me&lt;/Text&gt;
      &lt;/View&gt;
    &lt;/View&gt;
  );
}</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> React Native uses Flexbox for layout (just like CSS). The default flexDirection is 'column' (unlike 'row' in web CSS). Use <code>flex: 1</code> to make components fill available space.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">Q10: BASIC NAVIGATION</span>
  <h3 style="margin-top: 0;">How do you handle basic navigation between screens in React Native?</h3>

  <p>React Native doesn't include navigation by default. You typically use <strong>React Navigation</strong>, the most popular navigation library.</p>

  <div class="code-example">Basic Stack Navigation</div>
  <pre class="code-block"><code class="language-javascript">// Installation: npm install @react-navigation/native @react-navigation/stack
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';

const Stack = createStackNavigator();

function App() {
  return (
    &lt;NavigationContainer&gt;
      &lt;Stack.Navigator&gt;
        &lt;Stack.Screen name="Home" component={HomeScreen} /&gt;
        &lt;Stack.Screen name="Profile" component={ProfileScreen} /&gt;
      &lt;/Stack.Navigator&gt;
    &lt;/NavigationContainer&gt;
  );
}

// In a component, navigate programmatically
function HomeScreen({ navigation }) {
  return (
    &lt;View&gt;
      &lt;TouchableOpacity onPress={() =&gt; navigation.navigate('Profile')}&gt;
        &lt;Text&gt;Go to Profile&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/View&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">Q11: FLATLIST VS SCROLLVIEW</span>
  <h3 style="margin-top: 0;">What's the difference between FlatList and ScrollView?</h3>

  <p><strong>FlatList</strong> and <strong>ScrollView</strong> both enable scrolling, but they're optimized for different use cases:</p>

  <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 24px; margin: 24px 0;">
    <div style="background: #f0fdf4; border: 1px solid #bbf7d0; padding: 20px; border-radius: 12px;">
      <h4 style="margin: 0 0 12px 0; color: #166534;">📜 ScrollView</h4>
      <ul style="margin: 0; padding-left: 20px;">
        <li>Renders all items at once</li>
        <li>Best for small, fixed content</li>
        <li>Simple to use</li>
        <li>Can cause performance issues with many items</li>
      </ul>
    </div>
    <div style="background: #eff6ff; border: 1px solid #dbeafe; padding: 20px; border-radius: 12px;">
      <h4 style="margin: 0 0 12px 0; color: #1e40af;">📋 FlatList</h4>
      <ul style="margin: 0; padding-left: 20px;">
        <li>Renders items lazily (only visible ones)</li>
        <li>Best for large/dynamic lists</li>
        <li>More complex but performant</li>
        <li>Handles 1000+ items smoothly</li>
      </ul>
    </div>
  </div>

  <div class="code-example">FlatList Basic Usage</div>
  <pre class="code-block"><code class="language-javascript">import { FlatList } from 'react-native';

const data = [{id: 1, title: 'Item 1'}, {id: 2, title: 'Item 2'}];

function MyList() {
  return (
    &lt;FlatList
      data={data}
      keyExtractor={(item) =&gt; item.id.toString()}
      renderItem={({ item }) =&gt; (
        &lt;Text style={{padding: 20, borderBottomWidth: 1}}&gt;
          {item.title}
        &lt;/Text&gt;
      )}
    /&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">Q12: PLATFORM SPECIFIC CODE</span>
  <h3 style="margin-top: 0;">How do you write platform-specific code in React Native?</h3>

  <p>React Native provides several ways to handle platform differences:</p>

  <div class="code-example">Platform-Specific Code Patterns</div>
  <pre class="code-block"><code class="language-javascript">// 1. Platform module
import { Platform, Text } from 'react-native';

function MyComponent() {
  return (
    &lt;Text&gt;
      Running on {Platform.OS === 'ios' ? 'iOS' : 'Android'}
    &lt;/Text&gt;
  );
}

// 2. Platform-specific file extensions
// MyComponent.ios.js (iOS only)
// MyComponent.android.js (Android only)
// MyComponent.js (fallback for both)

// 3. Platform.select()
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  container: {
    ...Platform.select({
      ios: {
        paddingTop: 20,
      },
      android: {
        paddingTop: 0,
      },
    }),
  },
});</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Avoid platform-specific code when possible. React Native components adapt automatically to each platform. Only use platform-specific code for truly platform-unique features.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">Q13: REACT NATIVE CLI VS EXPO</span>
  <h3 style="margin-top: 0;">What's the difference between React Native CLI and Expo?</h3>

  <p><strong>Expo</strong> is a managed platform that simplifies React Native development, while <strong>React Native CLI</strong> gives you full control over the native layers.</p>

  <table style="width: 100%; border-collapse: collapse; margin: 24px 0;">
    <tr style="background: #f1f5f9;">
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Feature</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Expo</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">React Native CLI</th>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><strong>Setup</strong></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Very easy (5 minutes)</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Complex (Xcode/Android Studio needed)</td>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><strong>Native Code</strong></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Limited access</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Full access</td>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><strong>Publishing</strong></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Expo servers</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">App Store/Play Store</td>
    </tr>
    <tr>
      <td style="padding: 12px;"><strong>Best For</strong></td>
      <td style="padding: 12px;">Prototyping, learning</td>
      <td style="padding: 12px;">Production apps, custom native features</td>
    </tr>
  </table>
</div>

<div class="qa-card">
  <span class="question-tag">Q14: HANDLING USER INPUT</span>
  <h3 style="margin-top: 0;">How do you handle user input and form validation in React Native?</h3>

  <p>User input is primarily handled through the <code>TextInput</code> component with controlled state management.</p>

  <div class="code-example">TextInput with Validation</div>
  <pre class="code-block"><code class="language-javascript">import { useState } from 'react';
import { View, TextInput, Text, Alert } from 'react-native';

function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState({});

  const validateAndSubmit = () => {
    const newErrors = {};

    if (!email.includes('@')) {
      newErrors.email = 'Invalid email';
    }
    if (password.length &lt; 6) {
      newErrors.password = 'Password too short';
    }

    if (Object.keys(newErrors).length === 0) {
      // Submit form
      Alert.alert('Success', 'Form submitted!');
    } else {
      setErrors(newErrors);
    }
  };

  return (
    &lt;View style={{padding: 20}}&gt;
      &lt;TextInput
        placeholder="Email"
        value={email}
        onChangeText={setEmail}
        keyboardType="email-address"
        autoCapitalize="none"
        style={{
          borderWidth: 1,
          padding: 10,
          marginBottom: 10,
          borderColor: errors.email ? 'red' : '#ccc'
        }}
      /&gt;
      {errors.email && &lt;Text style={{color: 'red'}}&gt;{errors.email}&lt;/Text&gt;}

      &lt;TextInput
        placeholder="Password"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
        style={{
          borderWidth: 1,
          padding: 10,
          marginBottom: 10,
          borderColor: errors.password ? 'red' : '#ccc'
        }}
      /&gt;
      {errors.password && &lt;Text style={{color: 'red'}}&gt;{errors.password}&lt;/Text&gt;}
    &lt;/View&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">Q15: ERROR HANDLING</span>
  <h3 style="margin-top: 0;">How do you handle errors and loading states in React Native?</h3>

  <p>Error handling in React Native follows similar patterns to React web applications, using state to manage loading and error states.</p>

  <div class="code-example">Error and Loading States</div>
  <pre class="code-block"><code class="language-javascript">import { useState, useEffect } from 'react';
import { View, Text, ActivityIndicator, TouchableOpacity } from 'react-native';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const fetchUser = async () => {
    try {
      setLoading(true);
      setError(null);
      const response = await fetch(`https://api.example.com/users/${userId}`);
      if (!response.ok) throw new Error('Failed to fetch user');
      const userData = await response.json();
      setUser(userData);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() =&gt; {
    fetchUser();
  }, [userId]);

  if (loading) {
    return (
      &lt;View style={{flex: 1, justifyContent: 'center', alignItems: 'center'}}&gt;
        &lt;ActivityIndicator size="large" color="#007AFF" /&gt;
        &lt;Text style={{marginTop: 10}}&gt;Loading...&lt;/Text&gt;
      &lt;/View&gt;
    );
  }

  if (error) {
    return (
      &lt;View style={{flex: 1, justifyContent: 'center', alignItems: 'center', padding: 20}}&gt;
        &lt;Text style={{color: 'red', textAlign: 'center', marginBottom: 20}}&gt;
          Error: {error}
        &lt;/Text&gt;
        &lt;TouchableOpacity
          style={{backgroundColor: '#007AFF', padding: 10, borderRadius: 5}}
          onPress={fetchUser}
        &gt;
          &lt;Text style={{color: 'white'}}&gt;Retry&lt;/Text&gt;
        &lt;/TouchableOpacity&gt;
      &lt;/View&gt;
    );
  }

  return (
    &lt;View style={{padding: 20}}&gt;
      &lt;Text style={{fontSize: 24}}&gt;{user.name}&lt;/Text&gt;
      &lt;Text&gt;{user.email}&lt;/Text&gt;
    &lt;/View&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">Q16: TOUCHABLE COMPONENTS</span>
  <h3 style="margin-top: 0;">How do you handle touch interactions in React Native?</h3>

  <p>React Native provides several touchable components for handling user interactions. Unlike web buttons, these components provide native touch feedback.</p>

  <div class="component-grid">
    <div class="component-card">
      <span style="font-size: 1.5rem; font-weight: 800; color: #059669;">👆</span>
      <span style="display: block; font-weight: 600; margin: 8px 0;">TouchableOpacity</span>
      <span style="display: block; font-size: 0.875rem; color: #64748b;">Reduces opacity on press</span>
    </div>
    <div class="component-card">
      <span style="font-size: 1.5rem; font-weight: 800; color: #059669;">🔘</span>
      <span style="display: block; font-weight: 600; margin: 8px 0;">TouchableHighlight</span>
      <span style="display: block; font-size: 0.875rem; color: #64748b;">Changes background on press</span>
    </div>
    <div class="component-card">
      <span style="font-size: 1.5rem; font-weight: 800; color: #059669;">❌</span>
      <span style="display: block; font-weight: 600; margin: 8px 0;">TouchableWithoutFeedback</span>
      <span style="display: block; font-size: 0.875rem; color: #64748b;">No visual feedback</span>
    </div>
    <div class="component-card">
      <span style="font-size: 1.5rem; font-weight: 800; color: #059669;">🌊</span>
      <span style="display: block; font-weight: 600; margin: 8px 0;">Pressable</span>
      <span style="display: block; font-size: 0.875rem; color: #64748b;">Modern, flexible touchable</span>
    </div>
  </div>

  <div class="code-example">TouchableOpacity Example</div>
  <pre class="code-block"><code class="language-javascript">import { TouchableOpacity, Text, Alert } from 'react-native';

function TouchableExample() {
  const handlePress = () => {
    Alert.alert('Button Pressed!', 'You tapped the button');
  };

  return (
    &lt;TouchableOpacity
      style={{
        backgroundColor: '#007AFF',
        padding: 12,
        borderRadius: 8,
        alignItems: 'center'
      }}
      onPress={handlePress}
      activeOpacity={0.7} // Opacity when pressed
    &gt;
      &lt;Text style={{ color: 'white', fontSize: 16 }}&gt;
        Tap Me!
      &lt;/Text&gt;
    &lt;/TouchableOpacity&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">Q17: BASIC FLEXBOX</span>
  <h3 style="margin-top: 0;">How does Flexbox work in React Native for basic layouts?</h3>

  <p>React Native uses Flexbox for layout, but with some differences from web CSS. The default direction is <code>column</code> (not <code>row</code>), and you must specify dimensions.</p>

  <div class="code-example">Basic Flexbox Layouts</div>
  <pre class="code-block"><code class="language-javascript">// Vertical layout (default)
&lt;View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}&gt;
  &lt;Text&gt;Top&lt;/Text&gt;
  &lt;Text&gt;Middle&lt;/Text&gt;
  &lt;Text&gt;Bottom&lt;/Text&gt;
&lt;/View&gt;

// Horizontal layout
&lt;View style={{ flexDirection: 'row', justifyContent: 'space-between' }}&gt;
  &lt;Text&gt;Left&lt;/Text&gt;
  &lt;Text&gt;Right&lt;/Text&gt;
&lt;/View&gt;

// Equal width columns
&lt;View style={{ flexDirection: 'row' }}&gt;
  &lt;View style={{ flex: 1, backgroundColor: 'red' }} /&gt;
  &lt;View style={{ flex: 1, backgroundColor: 'blue' }} /&gt;
  &lt;View style={{ flex: 1, backgroundColor: 'green' }} /&gt;
&lt;/View&gt;</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> <code>flex: 1</code> means "take up all available space." Use it on containers to make them fill the screen. <code>justifyContent</code> controls main axis spacing, <code>alignItems</code> controls cross axis alignment.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">Q18: CONDITIONAL RENDERING</span>
  <h3 style="margin-top: 0;">How do you conditionally render components in React Native?</h3>

  <p>Conditional rendering works the same as in React - use JavaScript conditional operators to show/hide components based on state or props.</p>

  <div class="code-example">Conditional Rendering Patterns</div>
  <pre class="code-block"><code class="language-javascript">function ConditionalExample({ isLoggedIn, userName }) {
  // Method 1: Ternary operator
  return (
    &lt;View&gt;
      {isLoggedIn ? (
        &lt;Text&gt;Welcome back, {userName}!&lt;/Text&gt;
      ) : (
        &lt;Text&gt;Please log in&lt;/Text&gt;
      )}
    &lt;/View&gt;
  );
}

// Method 2: Logical AND (for simple cases)
function LoadingExample({ isLoading, data }) {
  return (
    &lt;View&gt;
      {isLoading && &lt;Text&gt;Loading...&lt;/Text&gt;}
      {!isLoading && data && &lt;Text&gt;Data loaded!&lt;/Text&gt;}
    &lt;/View&gt;
  );
}

// Method 3: Early return
function AuthGuard({ isAuthenticated, children }) {
  if (!isAuthenticated) {
    return &lt;Text&gt;Access denied&lt;/Text&gt;;
  }

  return &lt;View&gt;{children}&lt;/View&gt;;
}</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Always use conditional rendering instead of hiding components with <code>display: 'none'</code> or <code>opacity: 0</code>. Hidden components still render and take up memory.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">Q19: BASIC NAVIGATION CONCEPTS</span>
  <h3 style="margin-top: 0;">What are the basic concepts of navigation in React Native?</h3>

  <p>Navigation in React Native is handled by libraries like React Navigation. It manages moving between different screens and maintaining navigation history.</p>

  <div style="background: #f0fdf4; border: 1px solid #bbf7d0; padding: 16px; border-radius: 8px; margin: 16px 0;">
    <strong>📱 Navigation Basics:</strong>
    <ul style="margin: 8px 0 0 0; padding-left: 20px;">
      <li><strong>Stack Navigator:</strong> Screens stack on top of each other (like browser history)</li>
      <li><strong>Tab Navigator:</strong> Bottom tabs for switching between main sections</li>
      <li><strong>Drawer Navigator:</strong> Side menu that slides in from the edge</li>
      <li><strong>Navigation Prop:</strong> Every screen gets a <code>navigation</code> prop for moving between screens</li>
    </ul>
  </div>

  <div class="code-example">Basic Navigation Usage</div>
  <pre class="code-block"><code class="language-javascript">// In a screen component
function HomeScreen({ navigation }) {
  return (
    &lt;View&gt;
      &lt;TouchableOpacity onPress={() =&gt; navigation.navigate('Profile')}&gt;
        &lt;Text&gt;Go to Profile&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;

      &lt;TouchableOpacity onPress={() =&gt; navigation.goBack()}&gt;
        &lt;Text&gt;Go Back&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/View&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">Q20: BASIC API CALLS</span>
  <h3 style="margin-top: 0;">How do you make basic API calls in React Native?</h3>

  <p>React Native uses the same <code>fetch</code> API as modern browsers for making HTTP requests. Always handle loading states and errors.</p>

  <div class="code-example">Basic API Call with Loading State</div>
  <pre class="code-block"><code class="language-javascript">import { useState, useEffect } from 'react';
import { View, Text, ActivityIndicator } from 'react-native';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch(`https://jsonplaceholder.typicode.com/users/${userId}`)
      .then(response => {
        if (!response.ok) {
          throw new Error('Failed to fetch user');
        }
        return response.json();
      })
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, [userId]);

  if (loading) {
    return &lt;ActivityIndicator size="large" color="#007AFF" /&gt;;
  }

  if (error) {
    return &lt;Text style={{color: 'red'}}&gt;Error: {error}&lt;/Text&gt;;
  }

  return (
    &lt;View&gt;
      &lt;Text&gt;Name: {user.name}&lt;/Text&gt;
      &lt;Text&gt;Email: {user.email}&lt;/Text&gt;
    &lt;/View&gt;
  );
}</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Always handle the loading and error states when making API calls. Users should know when something is happening and when something went wrong.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">Q21: BASIC ERROR HANDLING</span>
  <h3 style="margin-top: 0;">How do you handle basic errors in React Native?</h3>

  <p>Error handling in React Native combines JavaScript try/catch with UI feedback to users. Always provide meaningful error messages and recovery options.</p>

  <div class="code-example">Error Handling Patterns</div>
  <pre class="code-block"><code class="language-javascript">// 1. Try/catch for synchronous errors
function SafeComponent() {
  const [data, setData] = useState(null);

  const loadData = () => {
    try {
      const result = riskyOperation();
      setData(result);
    } catch (error) {
      Alert.alert('Error', error.message);
    }
  };

  // 2. Async error handling
  const fetchData = async () => {
    try {
      setLoading(true);
      const response = await fetch('/api/data');
      if (!response.ok) {
        throw new Error('Network request failed');
      }
      const json = await response.json();
      setData(json);
    } catch (error) {
      setError(error.message);
    } finally {
      setLoading(false);
    }
  };

  // 3. Error boundaries (advanced)
  class ErrorBoundary extends Component {
    constructor(props) {
      super(props);
      this.state = { hasError: false };
    }

    static getDerivedStateFromError(error) {
      return { hasError: true };
    }

    render() {
      if (this.state.hasError) {
        return &lt;Text&gt;Something went wrong.&lt;/Text&gt;;
      }
      return this.props.children;
    }
  }</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">Q22: REACT NATIVE CLI VS EXPO</span>
  <h3 style="margin-top: 0;">What's the difference between React Native CLI and Expo?</h3>

  <p><strong>Expo</strong> is a managed platform that simplifies React Native development, while <strong>React Native CLI</strong> gives you full control over the native layers.</p>

  <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin: 24px 0;">
    <div style="background: #eff6ff; border: 1px solid #dbeafe; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #1e40af;">🚀 Expo (Beginner Friendly)</h4>
      <ul style="margin: 0; padding-left: 20px; font-size: 0.875rem;">
        <li>No Xcode/Android Studio needed</li>
        <li>Built-in services (push notifications, etc.)</li>
        <li>Easy testing on real devices</li>
        <li>Limited native module access</li>
      </ul>
    </div>
    <div style="background: #fef3c7; border: 1px solid #fde68a; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #92400e;">⚙️ React Native CLI (Full Control)</h4>
      <ul style="margin: 0; padding-left: 20px; font-size: 0.875rem;">
        <li>Requires native development setup</li>
        <li>Full access to native code</li>
        <li>Can use any native libraries</li>
        <li>More complex setup</li>
      </ul>
    </div>
  </div>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Start with Expo if you're new to React Native - it's much easier to get started. Switch to CLI later when you need advanced native features.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">Q23: BASIC PERFORMANCE TIPS</span>
  <h3 style="margin-top: 0;">What are some basic performance tips for React Native beginners?</h3>

  <p>Good performance habits start with the basics. Here are essential tips every React Native developer should know:</p>

  <div style="background: #f0fdf4; border: 1px solid #bbf7d0; padding: 16px; border-radius: 8px; margin: 16px 0;">
    <strong>⚡ Essential Performance Tips:</strong>
    <ul style="margin: 8px 0 0 0; padding-left: 20px;">
      <li><strong>Keys in Lists:</strong> Always provide <code>key</code> props for list items</li>
      <li><strong>Avoid Inline Functions:</strong> Don't create functions inside render</li>
      <li><strong>Use Memo:</strong> Wrap expensive components with <code>React.memo</code></li>
      <li><strong>Optimize Images:</strong> Compress images and use appropriate sizes</li>
      <li><strong>Minimize Renders:</strong> Use <code>useCallback</code> and <code>useMemo</code> for expensive operations</li>
    </ul>
  </div>

  <div class="code-example">Performance Anti-patterns to Avoid</div>
  <pre class="code-block"><code class="language-javascript">// ❌ BAD: Inline function in render
&lt;TouchableOpacity onPress={() =&gt; handlePress(item.id)}&gt;

// ✅ GOOD: Memoized callback
const handlePress = useCallback((id) => {
  // handle press
}, []);

// ❌ BAD: No keys in FlatList
&lt;FlatList data={items} renderItem={({item}) =&gt; &lt;Text&gt;{item}&lt;/Text&gt;} /&gt;

// ✅ GOOD: Always provide keys
&lt;FlatList
  data={items}
  keyExtractor={(item) => item.id.toString()}
  renderItem={({item}) =&gt; &lt;Text&gt;{item.name}&lt;/Text&gt;}
/&gt;</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">Q24: DEBUGGING BASICS</span>
  <h3 style="margin-top: 0;">How do you debug basic issues in React Native?</h3>

  <p>Debugging React Native involves both JavaScript and native debugging tools. Start with console logging and work up to advanced tools.</p>

  <div style="background: #fef3c7; border: 1px solid #fde68a; padding: 16px; border-radius: 8px; margin: 16px 0;">
    <strong>🐛 Basic Debugging Steps:</strong>
    <ol style="margin: 8px 0 0 0; padding-left: 20px;">
      <li><strong>Console Logs:</strong> Use <code>console.log()</code> to check values</li>
      <li><strong>Network Tab:</strong> Check API calls in browser dev tools</li>
      <li><strong>Component Inspector:</strong> Use React DevTools to inspect components</li>
      <li><strong>Hot Reload:</strong> Make changes and see them instantly</li>
      <li><strong>Breakpoints:</strong> Set debugger statements in code</li>
    </ol>
  </div>

  <div class="code-example">Basic Debugging</div>
  <pre class="code-block"><code class="language-javascript">// Console logging
function MyComponent({ data }) {
  console.log('Data received:', data); // Check what data looks like

  if (!data) {
    console.warn('No data provided to MyComponent'); // Warn about issues
    return null;
  }

  // Debugger breakpoint
  debugger; // Code will pause here in debugger

  return &lt;Text&gt;{data.name}&lt;/Text&gt;;
}

// Error boundaries for catching crashes
class ErrorBoundary extends Component {
  componentDidCatch(error, errorInfo) {
    console.error('Error caught:', error, errorInfo);
    // Report error to service
  }

  render() {
    if (this.state.hasError) {
      return &lt;Text&gt;Something went wrong&lt;/Text&gt;;
    }
    return this.props.children;
  }
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">Q25: COMMON BEGINNER MISTAKES</span>
  <h3 style="margin-top: 0;">What are common mistakes React Native beginners make?</h3>

  <p>Avoiding these common pitfalls will save you hours of debugging and frustration:</p>

  <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 16px; margin: 24px 0;">
    <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #dc2626;">❌ Text Outside Text Component</h4>
      <p style="margin: 0; font-size: 0.875rem;">Can't put text directly in View - always wrap in Text component.</p>
    </div>
    <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #dc2626;">❌ No Dimensions on Flex Items</h4>
      <p style="margin: 0; font-size: 0.875rem;">Flex items need explicit width/height or flex properties.</p>
    </div>
    <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #dc2626;">❌ Mutating State Directly</h4>
      <p style="margin: 0; font-size: 0.875rem;">Never do <code>state.count = 5</code> - always use setState/setter functions.</p>
    </div>
    <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #dc2626;">❌ Ignoring Platform Differences</h4>
      <p style="margin: 0; font-size: 0.875rem;">iOS and Android behave differently - test on both platforms.</p>
    </div>
  </div>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> The React Native documentation is excellent. When stuck, check the official docs first - they're comprehensive and up-to-date.
  </div>
</div>

---

<div class="nav-footer">
  <a href="../README.md" style="color: #64748b; text-decoration: none; font-weight: 600;">⬅️ Home</a>
  <a href="phase2-react-fundamentals.md" style="color: #2563eb; text-decoration: none; font-weight: 600;">Phase 02: React Fundamentals ➡️</a>
</div>

</div>
