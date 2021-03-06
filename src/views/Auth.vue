<template>
    <Header
        ref="header"
        title="AlekEagle ShareX Server"
        subtitle="Welcome back!"
        :buttons="[
            {
                text: 'I don\'t have an account',
                action: switchBetweenLoginAndRegister
            }
        ]"
    />

    <div class="projects">
        <Project
            title="Login"
            :noSpan="true"
            :classes="['float', 'auth']"
            v-if="hasAccount"
        >
            <form @submit.prevent="login">
                <input
                    class="darktextbox"
                    type="text"
                    autocomplete="username"
                    name="name"
                    maxlength="30"
                    placeholder="Username"
                    required
                />
                <input
                    class="darktextbox"
                    type="password"
                    autocomplete="password"
                    name="password"
                    maxlength="30"
                    placeholder="Password"
                    required
                />
                <button class="button">Login</button>
            </form>
        </Project>
        <Project
            title="Register"
            :noSpan="true"
            :classes="['float', 'auth']"
            v-if="!hasAccount"
        >
            <form @submit.prevent="register">
                <input
                    class="darktextbox"
                    type="text"
                    autocomplete="username"
                    name="name"
                    maxlength="30"
                    placeholder="Username"
                    required
                />
                <input
                    class="darktextbox"
                    type="email"
                    autocomplete="email"
                    name="email"
                    maxlength="30"
                    placeholder="E-mail"
                    required
                />
                <input
                    class="darktextbox"
                    type="password"
                    autocomplete="new-password"
                    name="password"
                    maxlength="30"
                    placeholder="Password"
                    required
                />
                <input
                    class="darktextbox"
                    type="password"
                    autocomplete="new-password"
                    name="password-conf"
                    maxlength="30"
                    placeholder="Confirm Password"
                    required
                />
                <button class="button">Register</button>
            </form>
        </Project>
    </div>
    <Footer />
</template>

<script>
import Project from '@/components/Project.vue';
import Header from '@/components/Header.vue';
import Footer from '@/components/Footer.vue';
function parseQueryString(qs, sep, eq) {
    let parsed = {};
    qs.split(sep || '&').forEach(p => {
        let ps = p.split(eq || '=');
        parsed[unescape(ps[0])] = ps[1] ? unescape(ps[1]) : true;
    });
    return parsed;
}
export default {
    title: 'Login',
    name: 'Auth',
    components: {
        Header,
        Project,
        Footer
    },
    methods: {
        switchBetweenLoginAndRegister() {
            this.hasAccount = !this.hasAccount;
            if (!this.hasAccount) {
                this.$refs.header.sharedState.buttons[0].text =
                    'I already have an account';
                this.$refs.header.sharedState.subtitle =
                    'Hi, hope you enjoy your stay!';
                document.title = 'Register';
            } else {
                this.$refs.header.sharedState.buttons[0].text =
                    "I don't have an account";
                this.$refs.header.sharedState.subtitle = 'Welcome back!';
                document.title = 'Login';
            }
        },
        login(e) {
            let data = new FormData(e.target);
            fetch('/api/login/', {
                headers: {
                    Authorization: window.localStorage.getItem('token')
                },
                method: 'POST',
                body: data
            }).then(
                res => {
                    switch (res.status) {
                        case 200:
                            res.json().then(json => {
                                window.localStorage.setItem(
                                    'token',
                                    json.apiToken
                                );
                                this.$router.push(
                                    window.location.href.split('?').length > 1
                                        ? parseQueryString(
                                              window.location.href.split('?')[1]
                                          ).redirect
                                        : '/me/'
                                );
                            });
                            break;
                        case 401:
                            res.json().then(json => {
                                this.$parent.$parent.temporaryToast(json.error);
                            });
                            break;
                        case 404:
                            res.json().then(json => {
                                this.$parent.$parent.temporaryToast(json.error);
                            });
                            break;
                        case 500:
                            res.json().then(json => {
                                this.$parent.$parent.temporaryToast(json.error);
                            });
                            break;
                        case 400:
                            this.$parent.$parent.temporaryToast(
                                'Please fill out all fields.'
                            );
                            break;
                        case 429:
                            this.$parent.$parent.temporaryToast(
                                `Woah, slow down! Please wait ${Math.floor(
                                    (res.headers.get('x-ratelimit-reset') *
                                        1000 -
                                        Date.now()) /
                                        1000 /
                                        60
                                )} minutes ${
                                    Math.floor(
                                        ((res.headers.get('x-ratelimit-reset') *
                                            1000 -
                                            Date.now()) /
                                            1000) %
                                            60
                                    ) !== 0
                                        ? `and ${Math.floor(
                                              ((res.headers.get(
                                                  'x-ratelimit-reset'
                                              ) *
                                                  1000 -
                                                  Date.now()) /
                                                  1000) %
                                                  60
                                          )} seconds`
                                        : ''
                                } before trying again!`
                            );
                            break;
                        default:
                            this.$parent.$parent.temporaryToast(
                                'An unknown error occurred, if this issue persists contact AlekEagle.',
                                10 * 1000
                            );
                    }
                },
                err => {
                    this.$parent.$parent.temporaryToast(
                        'An unknown error occurred, if this issue persists contact AlekEagle.',
                        10 * 1000
                    );
                    console.error(err);
                }
            );
        },
        register(e) {
            let data = new FormData(e.target);
            if (data.get('password') !== data.get('password-conf')) {
                this.$parent.$parent.temporaryToast(
                    "These passwords don't match!"
                );
                return;
            }
            data.delete('password-conf');
            fetch('/api/user/', {
                headers: {
                    Authorization: window.localStorage.getItem('token')
                },
                method: 'POST',
                body: data
            }).then(
                res => {
                    switch (res.status) {
                        case 201:
                            res.json().then(json => {
                                window.localStorage.setItem(
                                    'token',
                                    json.apiToken
                                );
                                this.$router.push(
                                    window.location.href.split('?').length > 1
                                        ? parseQueryString(
                                              window.location.href.split('?')[1]
                                          ).redirect
                                        : '/me/'
                                );
                            });
                            break;
                        case 401:
                            this.$parent.$parent.temporaryToast(
                                'There is already an account with this username or email.'
                            );
                            break;
                        case 429:
                            this.$parent.$parent.temporaryToast(
                                `Woah, slow down! Please wait ${Math.floor(
                                    (res.headers.get('x-ratelimit-reset') *
                                        1000 -
                                        Date.now()) /
                                        1000 /
                                        60
                                )} minutes ${
                                    Math.floor(
                                        ((res.headers.get('x-ratelimit-reset') *
                                            1000 -
                                            Date.now()) /
                                            1000) %
                                            60
                                    ) !== 0
                                        ? `and ${Math.floor(
                                              ((res.headers.get(
                                                  'x-ratelimit-reset'
                                              ) *
                                                  1000 -
                                                  Date.now()) /
                                                  1000) %
                                                  60
                                          )} seconds`
                                        : ''
                                } before trying again!`
                            );
                            break;
                        default:
                            this.$parent.$parent.temporaryToast(
                                'An unknown error occurred, if this issue persists contact AlekEagle.',
                                10 * 1000
                            );
                    }
                },
                err => {
                    this.$parent.$parent.temporaryToast(
                        'An unknown error occurred, if this issue persists contact AlekEagle.',
                        10 * 1000
                    );
                    console.error(err);
                }
            );
        }
    },
    data() {
        return {
            hasAccount: true
        };
    },
    beforeMount() {
        fetch('/api/user/', {
            headers: {
                Authorization: `${window.localStorage.getItem('token')}`
            }
        }).then(
            res => {
                if (res.status === 200) {
                    this.$router.push(
                        window.location.href.split('?').length > 1
                            ? parseQueryString(
                                  window.location.href.split('?')[1]
                              ).redirect
                            : '/me/'
                    );
                } else {
                    window.localStorage.removeItem('token');
                }
            },
            err => {
                this.$parent.$parent.temporaryToast(
                    'An unknown error occurred, if this issue persists contact AlekEagle.',
                    10 * 1000
                );
                console.error(err);
            }
        );
    }
};
</script>
