# GitLab Enumeration

## Discovery & Footprinting

We can quickly determine that GitLab is in use in an environment by just browsing to the GitLab URL, and we will be directed to the login page, which displays the GitLab logo.

<figure><img src="../../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

### Version Discovery

The only way to footprint the GitLab version number in use is by browsing to the `/help` page when logged in. If the GitLab instance allows us to register an account, we can log in and browse to this page to confirm the version.

If we cannot register an account, we may have to try a low-risk exploit such as [this](https://www.exploit-db.com/exploits/49821).

{% hint style="warning" %}
## Undiscovered Version

If we have no way to enumerate the version number (such as a date on the page, the first public commit, or by registering a user), then we should stick to hunting for secrets and not try multiple exploits against it blindly.

**Do not blindly launch various exploits at an application**
{% endhint %}

***

## Enumeration

### Public Projects

There's not much we can do against GitLab without knowing the version number or being logged in. The first thing we should try is browsing to `/explore` and see if there are any public projects that may contain something interesting.

Public projects can be interesting because we may be able to use them to find out more about the company's infrastructure, find production code that we can find a bug in after a code review, hard-coded credentials, a script or configuration file containing credentials, or other secrets such as an SSH private key or API key.

### Credentialed Access

Once we are done digging through what is available externally, we should check and see if we can register an account and access additional projects.

Suppose the organisation did not set up GitLab only to allow company emails to register or require an admin to approve a new account. In that case, we may be able to access additional data.

***

## Enumerating Users

### Manual Enumeration

We can use the registration form to enumerate valid users. On this some instances of GitLab we can also enumerate emails. If we try to register with an email that has already been taken, we will get the error:&#x20;

`1 error prohibited this user from being saved: Email has already been taken`

As of the time of writing, this username enumeration technique works with the latest version of GitLab.

<figure><img src="../../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

Even if the `Sign-up enabled` checkbox is cleared within the settings page under `Sign-up restrictions`, we can still browse to the `/users/sign_up` page and enumerate users but will not be able to register a user.

{% hint style="info" %}
## Mitigations

Mitigations include enforcing 2FA on all user accounts, using `Fail2Ban` to block failed login attempts which are indicative of brute-forcing attacks, and even restricting which IP addresses can access a GitLab instance if it must be accessible outside of the internal corporate network.
{% endhint %}

### Automated Enumeration

We can use scripts like [this one](https://www.exploit-db.com/exploits/49821) to enumerate a list of valid users. The Python3 version of this same tool can be found [here](https://github.com/dpgg101/GitLabUserEnum). As with any type of password spraying attack, we should be mindful of account lockout and other kinds of interruptions.

GitLab's defaults are set to 10 failed login attempts, resulting in an automatic unlock after 10 minutes.

Starting with GitLab version 16.6, administrators can now configure these values directly through the admin UI. The number of authentication attempts before locking an account and the unlock period can be set using the `max_login_attempts` and `failed_login_attempts_unlock_period_in_minutes` settings, respectively. This configuration can be found [here](https://gitlab.com/gitlab-org/gitlab-foss/-/blob/master/config/initializers/8_devise.rb).

However, if these settings are not manually configured, they will still default to 10 failed login attempts and an unlock period of 10 minutes.

Additionally, while admins can modify the minimum password length to encourage stronger passwords, this alone will not fully mitigate the risk of password attacks.

{% hint style="info" %}
## User List

Use the Usernames provided in SecLists:

[https://github.com/danielmiessler/SecLists](https://github.com/danielmiessler/SecLists)
{% endhint %}

### gitlab\_userenum.py

{% code title="Download Command" %}
```bash
wget https://raw.githubusercontent.com/dpgg101/GitLabUserEnum/main/gitlab_userenum.py
```
{% endcode %}

{% code title="Execution" %}
```bash
python3 gitlab_userenum.py --url <URL> --wordlist /usr/share/wordlists/<USER_LIST>
```
{% endcode %}

***

## Advanced Git Enumeration

[https://tillsongalloway.com/finding-sensitive-information-on-github/index.html](https://tillsongalloway.com/finding-sensitive-information-on-github/index.html)
