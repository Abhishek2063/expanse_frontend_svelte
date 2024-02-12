<script>
  import { Route, Router, navigate } from "svelte-routing";
  import Home from "./views/home/home.svelte";
  import Login from "./views/login/login.svelte";
  import Register from "./views/register/register.svelte";
  import Dashboard from "./views/dashboard/dashboard.svelte";
  import { private_route, public_routes } from "./routes/app_routes";
  import { getToken } from "./utils/storage/token";
  import PublicLayout from "./layouts/PublicLayout.svelte";
  import PrivateLayout from "./layouts/PrivateLayout.svelte";
  import Income from "./views/income/income.svelte";
  import Expanse from "./views/expanse/expanse.svelte";
  import Profile from "./views/profile/profile.svelte";
  // Check if the access token exists in the session storage
  const accessToken = getToken();

  // Function to determine if the user is authenticated
  function isAuthenticated() {
    return !!accessToken;
  }

  // Function to check if the current route is a private route
  function isPrivateRoute(pathname) {
    return Object.values(private_route).includes(pathname);
  }

  // Function to check if the current route is a private route
  function isPublicRoute(pathname) {
    return Object.values(public_routes).includes(pathname);
  }

  // Redirect to the login page if trying to access a private route without authentication
  if (!isAuthenticated() && isPrivateRoute(window.location.pathname)) {
    navigate(public_routes.login);
  }

  // Redirect to the dashboard if authenticated and trying to access a public route
  if (isAuthenticated() && isPublicRoute(window.location.pathname)) {
    navigate(private_route.dashboard);
  }

  // If the route is not found in public routes and not authenticated, redirect to the home page
  if (!isAuthenticated() && !isPublicRoute(window.location.pathname)) {
    navigate(public_routes.home);
  }

  // If the route is not found in private routes and  authenticated, redirect to the dashboard page
  if (isAuthenticated() && !isPrivateRoute(window.location.pathname)) {
    navigate(private_route.dashboard);
  }
</script>

<Router>
  <Route path={private_route.dashboard} component={Dashboard} />
  <Route path={private_route.income} component={Income} />
  <Route path={private_route.expense} component={Expanse} />
  <Route path={private_route.profile} component={Profile} />
  <Route path={public_routes.login} component={Login} />
  <Route path={public_routes.register} component={Register} />
  <Route path={public_routes.home} component={Home} />
</Router>
