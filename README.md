const corsHeaders = {
    "Access-Control-Allow-Origin": "*",
    "Access-Control-Allow-Methods": "GET,HEAD,POST,OPTIONS",
    "Access-Control-Max-Age": "86400",
};

addEventListener('fetch', event => {
    event.respondWith(handle(event.request)
        .catch(e => new Response(null, {
            status: 500,
            statusText: "Internal Server Error"
        }))
    )
})

async function handle(request) {
    if (request.method === "OPTIONS") {
        return handleOptions()
    } else {
        let url = new URL(request.url)
        let requestUrl = request.url.replace(`${url.origin}/`, '');
        request = new Request(requestUrl, request)
        request.headers.set("Origin", new URL(requestUrl).origin);
        let response = await fetch(request);
        // Recreate the response so you can modify the headers

        response = new Response(response.body, response);
        // Set CORS headers

        response.headers.set("Access-Control-Allow-Origin", "*");
        response.headers.set("Access-Control-Allow-Methods", "GET,HEAD,POST,OPTIONS");
        response.headers.set("Access-Control-Max-Age", "86400");
        response.headers.set("Location", `${url.origin}/` + response.headers.get('Location'));

        // Append to/Add Vary header so browser will cache response correctly
        response.headers.append("Vary", "Origin");

        return response;
    }
}

async function handleOptions(request) {
    if (
        request.headers.get("Origin") !== null &&
        request.headers.get("Access-Control-Request-Method") !== null &&
        request.headers.get("Access-Control-Request-Headers") !== null
    ) {
        // Handle CORS preflight requests.
        return new Response(null, {
            headers: {
                ...corsHeaders,
                "Access-Control-Allow-Headers": request.headers.get(
                    "Access-Control-Request-Headers"
                ),
            },
        });
    }
}
