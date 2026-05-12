# Login Page â KKRI Style

Dark glass morphism login page with teal/cyan accents. This is the `index.html` file.

## Visual Identity

- **Theme**: Dark, moody, professional
- **Background**: Same bg.jpg image with a dark overlay (`rgba(20,55,75,.35)`)
- **Card**: `rgba(20,50,70,.75)`, `blur(20px)`, `border-radius: 24px`
- **Accent color**: Teal gradient `linear-gradient(135deg, #2a7d8e, #4db8c7)`
- **Input fields**: Dark transparent backgrounds with teal focus glow
- **Error messages**: Red-tinted with shake animation

## Complete Template

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My App â Login</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap"
        rel="stylesheet">
    <style>
        * { margin:0; padding:0; box-sizing:border-box; }

        body {
            font-family: 'Inter', system-ui, sans-serif;
            background: #1a3a4a;
            color: #e2e8f0;
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            overflow: hidden;
        }

        /* Background image with dark overlay */
        .bg-axia {
            position: fixed; inset: 0; z-index: 0;
            background: url('img/bg.jpg') center center / cover no-repeat;
        }
        .bg-axia::after {
            content: '';
            position: absolute; inset: 0;
            background: rgba(20, 55, 75, .35);
        }

        .login-container {
            position: relative; z-index: 10;
            width: 100%; max-width: 440px; padding: 20px;
        }

        .login-card {
            background: rgba(20, 50, 70, .75);
            backdrop-filter: blur(20px);
            -webkit-backdrop-filter: blur(20px);
            border: 1px solid rgba(90, 190, 210, .2);
            border-radius: 24px;
            padding: 48px 40px;
            box-shadow: 0 25px 80px rgba(0,0,0,.4), 0 0 60px rgba(56,180,210,.08);
            animation: cardAppear .8s cubic-bezier(.16,1,.3,1);
        }

        @keyframes cardAppear {
            from { opacity:0; transform:translateY(30px) scale(.95); }
            to { opacity:1; transform:translateY(0) scale(1); }
        }

        /* Logo section */
        .login-logo { text-align: center; margin-bottom: 32px; }
        .login-logo .icon {
            display: inline-flex; align-items: center; justify-content: center;
            width: 180px; height: auto; margin-bottom: 16px;
        }
        .login-logo .icon img {
            width: 100%; height: auto;
            filter: drop-shadow(0 4px 20px rgba(255,255,255,.15));
        }
        .login-logo h1 {
            font-size: 24px; font-weight: 800;
            background: linear-gradient(135deg, #f8fafc, #94a3b8);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            letter-spacing: -.5px;
        }
        .login-logo p { color: #64748b; font-size: 13px; margin-top: 4px; }

        /* Form fields */
        .form-group { margin-bottom: 20px; }
        .form-group label {
            display: block; font-size: 12px; font-weight: 600;
            color: #94a3b8; text-transform: uppercase;
            letter-spacing: .5px; margin-bottom: 6px;
        }
        .form-group input {
            width: 100%; padding: 12px 16px;
            background: rgba(15, 23, 42, .6);
            border: 1px solid rgba(51, 65, 85, .8);
            border-radius: 12px; color: #f1f5f9;
            font-size: 15px; font-family: 'Inter', sans-serif;
            transition: all .3s; outline: none;
        }
        .form-group input:focus {
            border-color: #4db8c7;
            box-shadow: 0 0 0 3px rgba(77, 184, 199, .15);
            background: rgba(15, 35, 50, .8);
        }
        .form-group input::placeholder { color: #475569; }

        /* Login button â teal gradient */
        .login-btn {
            width: 100%; padding: 14px;
            background: linear-gradient(135deg, #2a7d8e, #4db8c7);
            border: none; border-radius: 12px;
            color: #fff; font-size: 15px; font-weight: 700;
            cursor: pointer; transition: all .3s;
            font-family: 'Inter', sans-serif;
            letter-spacing: .5px; position: relative; overflow: hidden;
        }
        .login-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 30px rgba(42, 125, 142, .4);
        }
        .login-btn:active { transform: translateY(0); }
        .login-btn:disabled { opacity: .6; cursor: not-allowed; transform: none; }

        /* Loading spinner on button */
        .login-btn .spinner {
            display: none; width: 20px; height: 20px;
            border: 2px solid rgba(255,255,255,.3);
            border-top-color: #fff; border-radius: 50%;
            animation: spin .6s linear infinite; margin: 0 auto;
        }
        .login-btn.loading span { display: none; }
        .login-btn.loading .spinner { display: block; }
        @keyframes spin { to { transform: rotate(360deg); } }

        /* Error message */
        .error-msg {
            background: rgba(239, 68, 68, .1);
            border: 1px solid rgba(239, 68, 68, .3);
            color: #fca5a5; padding: 10px 14px;
            border-radius: 10px; font-size: 13px;
            margin-bottom: 16px; display: none;
            animation: shake .4s ease-in-out;
        }
        .error-msg.show { display: flex; align-items: center; gap: 8px; }
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            25% { transform: translateX(-6px); }
            75% { transform: translateX(6px); }
        }

        /* Footer */
        .login-footer {
            text-align: center; margin-top: 24px;
            font-size: 11px; color: #475569;
        }
        .login-footer a { color: #4db8c7; text-decoration: none; }

        /* First-time registration (hidden by default) */
        .register-section {
            display: none; margin-top: 20px; padding-top: 20px;
            border-top: 1px solid rgba(51, 65, 85, .5);
        }
        .register-section h3 { color: #4db8c7; font-size: 14px; margin-bottom: 12px; }
        .register-section.show { display: block; }
    </style>
</head>
<body>
    <div class="bg-axia"></div>

    <div class="login-container">
        <div class="login-card">
            <div class="login-logo">
                <div class="icon"><img src="logo.png" alt="App Logo"></div>
                <h1>My App</h1>
                <p>Application description</p>
            </div>

            <div class="error-msg" id="error-msg">
                <span>&#9888;</span>
                <span id="error-text"></span>
            </div>

            <form id="login-form" onsubmit="handleLogin(event)">
                <div class="form-group">
                    <label>Email</label>
                    <input type="email" id="login-email" placeholder="your@email.com"
                           required autocomplete="email">
                </div>
                <div class="form-group">
                    <label>Password</label>
                    <input type="password" id="login-password" placeholder="â¢â¢â¢â¢â¢â¢â¢â¢"
                           required autocomplete="current-password">
                </div>
                <button type="submit" class="login-btn" id="login-btn">
                    <span>Sign In</span>
                    <div class="spinner"></div>
                </button>
            </form>

            <!-- First-time admin registration (shown when no users exist) -->
            <div class="register-section" id="register-section">
                <h3>Create first admin account</h3>
                <form id="register-form" onsubmit="handleRegister(event)">
                    <div class="form-group">
                        <label>First Name</label>
                        <input type="text" id="reg-firstname" placeholder="First Name" required>
                    </div>
                    <div class="form-group">
                        <label>Last Name</label>
                        <input type="text" id="reg-lastname" placeholder="Last Name" required>
                    </div>
                    <div class="form-group">
                        <label>Email</label>
                        <input type="email" id="reg-email" placeholder="admin@company.com" required>
                    </div>
                    <div class="form-group">
                        <label>Password (8 chars min)</label>
                        <input type="password" id="reg-password" placeholder="â¢â¢â¢â¢â¢â¢â¢â¢"
                               required minlength="8">
                    </div>
                    <button type="submit" class="login-btn" id="reg-btn"
                            style="background:linear-gradient(135deg,#34d399,#2a7d8e)">
                        <span>Create admin account</span>
                        <div class="spinner"></div>
                    </button>
                </form>
            </div>

            <div class="login-footer">
                <p>&copy; 2026 â My App</p>
            </div>
        </div>
    </div>

    <script>
        const API = 'api/';

        async function apiCall(endpoint, options = {}) {
            const res = await fetch(API + endpoint, {
                credentials: 'same-origin',
                headers: { 'Content-Type': 'application/json', ...options.headers },
                ...options
            });
            return res.json();
        }

        function showError(msg) {
            const el = document.getElementById('error-msg');
            document.getElementById('error-text').textContent = msg;
            el.classList.add('show');
            setTimeout(() => el.classList.remove('show'), 5000);
        }

        // Auto-check auth on load
        (async () => {
            try {
                const data = await apiCall('auth.php?action=check');
                if (data.authenticated) {
                    window.location.href = 'app.html';
                    return;
                }
                if (data.needs_setup) {
                    document.getElementById('register-section').classList.add('show');
                }
            } catch (e) { }
        })();

        async function handleLogin(e) {
            e.preventDefault();
            const btn = document.getElementById('login-btn');
            btn.classList.add('loading');
            btn.disabled = true;
            try {
                const data = await apiCall('auth.php?action=login', {
                    method: 'POST',
                    body: JSON.stringify({
                        email: document.getElementById('login-email').value,
                        password: document.getElementById('login-password').value
                    })
                });
                if (data.success) {
                    window.location.href = 'app.html';
                } else {
                    showError(data.error || 'Login error');
                }
            } catch (err) {
                showError('Server connection error');
            } finally {
                btn.classList.remove('loading');
                btn.disabled = false;
            }
        }

        async function handleRegister(e) {
            e.preventDefault();
            const btn = document.getElementById('reg-btn');
            btn.classList.add('loading');
            btn.disabled = true;
            try {
                const data = await apiCall('auth.php?action=register', {
                    method: 'POST',
                    body: JSON.stringify({
                        prenom: document.getElementById('reg-firstname').value,
                        nom: document.getElementById('reg-lastname').value,
                        email: document.getElementById('reg-email').value,
                        password: document.getElementById('reg-password').value
                    })
                });
                if (data.success) {
                    // Auto-login after registration
                    document.getElementById('login-email').value =
                        document.getElementById('reg-email').value;
                    document.getElementById('login-password').value =
                        document.getElementById('reg-password').value;
                    document.getElementById('register-section').classList.remove('show');
                    handleLogin(new Event('submit'));
                } else {
                    showError(data.error || 'Registration error');
                }
            } catch (err) {
                showError('Server error');
            } finally {
                btn.classList.remove('loading');
                btn.disabled = false;
            }
        }
    </script>
</body>
</html>
```

## Key Design Decisions

- **No separate CSS file**: Login page is self-contained with inline styles. This way it loads
  instantly and doesn't depend on any external CSS file.
- **Auto-check auth**: On page load, checks if already authenticated â redirects to app.html
- **Auto-show register**: If the API returns `needs_setup: true`, the registration form appears.
  This handles first-time deployment gracefully.
- **Loading state on button**: The spinner replaces button text during API calls, preventing
  double-clicks and giving visual feedback.
- **Error shake animation**: Draws attention to errors with a brief horizontal shake.
- **Green gradient for register button**: Differentiates the registration action from login.
