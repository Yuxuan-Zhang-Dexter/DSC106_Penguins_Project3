<script>
    import { base } from '$app/paths';
    import App from '../components/App.svelte';
  </script>
  
  <nav>
    <a href="{base}/">Home</a>
    <a href="{base}/about">About</a>
  </nav>
  
  <App />