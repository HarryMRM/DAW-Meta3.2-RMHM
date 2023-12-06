<template>
    <div ref="googleLoginBtn" class="btn_google"></div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router';

const router = useRouter();

const googleLoginBtn = ref(null);
let token = null;
let tokenExpirationTimer = null;

onMounted(() => {
    window.google.accounts.id.initialize({
        client_id: "176211644853-fnua99b0b5dn52jfs7cru2ukip2b1cos.apps.googleusercontent.com",
        callback: onSuccess,
        context: 'signin',
        auto_select: false,
        scope: process.env.VUE_APP_GOOGLE_SCOPES,
        referrerPolicy: {
            policy: 'strict-origin-when-cross-origin'
        }
    });
    window.google.accounts.id.renderButton(
        googleLoginBtn.value, {
        text: 'signin_with', // or 'signup_with' | 'continue_with' | 'signin'
        size: 'large', // or 'small' | 'medium'
        width: '366', // max width 400
        theme: 'outline', // or 'filled_black' |  'filled_blue'
        logo_alignment: 'center' // or 'center'
    });
});

function parseJwt(token) {
    const base64Url = token.split('.')[1];
    const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
    /* Buffer.from(base64,'base64')
    decodeURIComponent(atob(base64).split('').map(function (c) */
    const jsonPayload = decodeURIComponent(atob(base64).split('').map(function (c) {
        return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2);
    }).join(''));

    return JSON.parse(jsonPayload);
}

function setTokenExpirationTimer(expirationTime) {
    const timeUntilExpiration = expirationTime * 1000;
    tokenExpirationTimer = setTimeout(() => {
        router.push('/Login');
        console.log('Token ha expirado');
    }, timeUntilExpiration);
}


async function onSuccess(googleUser) {
    // Enviar la credencial al backend usando fetch
    try {
        const response = await fetch('https://localhost:3000/auth/verificarToken', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({
                credential: googleUser.credential,
            }),
        });

        if (response.ok) {
            const responseData = await response.json();
            console.log('Respuesta del backend:', responseData);

            token = responseData.jwtToken;
            localStorage.setItem('jwtToken', token);
            const decodedToken = parseJwt(token);
            const expirationTime = decodedToken.expiresIn;

            
            if (tokenExpirationTimer) {
                clearTimeout(tokenExpirationTimer);
            }
            setTokenExpirationTimer(expirationTime);
            router.push('/TablaPersonas');
        } else {
            console.error('Error en la solicitud al backend:', response.status);
        }
    } catch (error) {
        console.error('Error al enviar la credencial al backend:', error);
    }
}
</script>
<style>
.btn_google {
    display: flex;
    justify-content: center;
}
</style>