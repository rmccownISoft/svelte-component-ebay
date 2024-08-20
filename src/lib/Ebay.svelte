<script lang="ts">
    import { onMount } from "svelte";

    let data: any = null;
    let error: string | null = null;
    let params: { [key: string]: string } = {};

    async function makeRandomToken() {
        try {
            const array = new Uint8Array(20);
            window.crypto.getRandomValues(array);
            return Array.from(array, (byte) =>
                byte.toString(16).padStart(2, "0"),
            ).join("");
        } catch (err) {
            throw err;
        }
    }

    async function generateUrl() {
        const key = "ebaySandboxRefreshToken";
        const production = false;
        // create and save token
        const token = await makeRandomToken();
        // return url string
        const base = `https://ebay-redirect-server-puq6ph6rnq-uc.a.run.app/signin?token=${token}`;
        //const base = `http://192.168.1.162:3000/signin?token=${token}`;
        //const base = `http://localhost:3000/signin?token=${token}`;
        return base + "&sandbox=true";
    }

    // const doFetchWithParams(params: params) {
    //     const queryString = new URLSearchParams(params).toString()
    //     const url = `https://api.example.com/data?${queryString}`;

    //     fetch('https://example.com?' + new URLSearchParams({
    //     foo: 'value',
    //     bar: 2,
    //     }).toString())
    // }
    async function fetchWithRedirect() {
        const url = await generateUrl();

        try {
            // Make a fetch request to the proxy server
            const response = await fetch(url, {
                method: "GET",
                redirect: "manual", // Prevent automatic redirect
            });

            // Check if the response is a redirect
            if (
                response.status === 301 ||
                response.status === 302 ||
                response.status === 303
            ) {
                // Get the 'Location' header from the response (redirect URL)
                const redirectUrl = response.headers.get("Location");
                console.log(`Redirecting to: ${redirectUrl}`);

                // Optionally, follow the redirect manually by making a second fetch request
                const redirectedResponse = await fetch(redirectUrl, {
                    method: "GET",
                    redirect: "follow", // Follow any subsequent redirects
                });

                const data = await redirectedResponse.json();
                console.log("Redirected Data:", data);
            } else {
                // Handle non-redirect response
                const data = await response.json();
                console.log("Data:", data);
            }
        } catch (error) {
            console.error("Error fetching data:", error);
        }
    }

    async function fetchSignInUrl() {
        try {
            const url = await generateUrl();
            console.log("generated url: ", url);
            const response = await fetch(url, {
                redirect: "follow",
            });
            //const response = await fetch(url);
            console.log("response status: ", response.status);
            if (response.status === 302 || response.status === 303) {
                const redirectUrl = response.headers.get("Location");
                console.log("response redirect url: ", redirectUrl);
                if (redirectUrl) {
                    window.open(redirectUrl, "_blank");
                } else {
                    throw new Error(
                        "Redirect URL not found in response headers",
                    );
                }
            } else if (!response.ok) {
                throw new Error("Network response was not ok");
            } else {
                data = await response.json();
            }
        } catch (err) {
            error = (err as Error).message;
        }
    }
</script>

<div>
    <button on:click={fetchWithRedirect}>Fetch Sign-In Url</button>
    {#if data}
        <pre>{JSON.stringify(data, null, 2)}</pre>
    {/if}
    {#if error}
        <p>Error: {error}</p>
    {/if}
</div>
