<script lang="ts">
    import Button from '#lib/button/Button.svelte';
    import Main from '#lib/Main.svelte';
    import ContentCenter from '#lib/ContentCenter.svelte';
    import { useI18n } from '#state/i18n.svelte.ts';
    import Template from '#lib/Template.svelte';
    import { TPL_ADMIN_BUTTON_HIDE, TPL_IS_REG_OPEN } from '#utils/constants.ts';
    import ThemeSwitch from '#lib/ThemeSwitch.svelte';
    import LangSelector from '#lib/LangSelector.svelte';
    import { onMount } from 'svelte';
    import { fetchGet } from '$api/fetch';
    import type { SessionInfoResponse } from '$api/types/session';
    import type { UserResponse } from '$api/types/user';
    import { redirectToLogin, redirectToLogout } from '$utils/helpers';

    const btnWidth = '12rem';

    let t = useI18n();
    let hideAdminButton = $state(true);
    let isRegOpen = $state(false);

    let session: SessionInfoResponse | undefined = $state(undefined);
    let user: UserResponse | undefined = $state(undefined);
    let isLoading = $state(true);

    onMount(async () => {
        const sessionRes = await fetchGet<SessionInfoResponse>('/auth/v1/oidc/sessioninfo', 'json', 'noRedirect');
        if (sessionRes.status === 200 && sessionRes.body && sessionRes.body.state === 'Auth') {
            session = sessionRes.body;
            if (session.user_id) {
                const userRes = await fetchGet<UserResponse>(`/auth/v1/users/${session.user_id}`, 'json', 'noRedirect');
                if (userRes.status === 200 && userRes.body) {
                    user = userRes.body;
                }
            } else {
                redirectToLogin('account');
                return;
            }
        } else {
            redirectToLogin('account');
            return;
        }
        isLoading = false;
    });

    function handleLogin() {
        redirectToLogin('account');
    }

    function handleLogout() {
        redirectToLogout();
    }

    function redirectToAdmin() {
        window.location.href = '/auth/v1/admin';
    }

    function redirectToAccount() {
        window.location.href = '/auth/v1/account';
    }

    function redirectToReg() {
        window.location.href = '/auth/v1/users/register';
    }
</script>

<svelte:head>
    <title>Rauthy Home</title>
</svelte:head>

<Template id={TPL_ADMIN_BUTTON_HIDE} bind:value={hideAdminButton} />
<Template id={TPL_IS_REG_OPEN} bind:value={isRegOpen} />

<Main>
    <ContentCenter>
        {#if !isLoading}
            <div class="container">
                {#if user}
                    <div class="user-info">
                        <span class="welcome-text">Logged in as:</span>
                        <span class="user-name">
                            {user.given_name || ''} {user.family_name || ''}
                        </span>
                        <span class="user-email">{user.email}</span>
                    </div>
                {/if}

                <div class="btn">
                    {#if user}
                        <Button ariaLabel={t.index.accountLogin} onclick={redirectToAccount} width={btnWidth}>
                            {t.index.accountLogin}
                        </Button>
                        {#if !hideAdminButton}
                            <Button
                                ariaLabel={t.index.adminLogin}
                                level={3}
                                onclick={redirectToAdmin}
                                width={btnWidth}
                            >
                                {t.index.adminLogin}
                            </Button>
                        {/if}
                        <Button ariaLabel={t.logout.logout} level={2} onclick={handleLogout} width={btnWidth}>
                            {t.logout.logout}
                        </Button>
                    {:else}
                        <Button ariaLabel={t.index.accountLogin} onclick={handleLogin} width={btnWidth}>
                            {t.index.accountLogin}
                        </Button>
                        {#if isRegOpen}
                            <Button
                                ariaLabel={t.index.register}
                                level={2}
                                onclick={redirectToReg}
                                width={btnWidth}
                            >
                                {t.index.register}
                            </Button>
                        {/if}
                    {/if}
                </div>
            </div>
        {/if}
    </ContentCenter>

    <ThemeSwitch absolute />
    <LangSelector absolute />
</Main>

<style>
    .btn {
        display: flex;
        flex-direction: column;
        gap: 0.5rem;
    }
    .container {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 1.5rem;
    }
    .user-info {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 0.25rem;
        background-color: var(--bg-card);
        padding: 1rem 1.5rem;
        border-radius: var(--border-radius);
        border: 1px solid var(--border-color);
        text-align: center;
    }
    .welcome-text {
        font-size: 0.85rem;
        color: var(--text-secondary);
    }
    .user-name {
        font-size: 1.25rem;
        font-weight: 600;
    }
    .user-email {
        font-size: 0.9rem;
        color: var(--text-secondary);
    }
</style>
