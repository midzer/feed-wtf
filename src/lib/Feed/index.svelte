<script>
    import './index.css';

    // urlB64ToUint8Array is a magic function that will encode the base64 public key
    // to Array buffer which is needed by the subscription option
    function urlB64ToUint8Array (base64String) {
        const padding = '='.repeat((4 - (base64String.length % 4)) % 4);
        const base64 = (base64String + padding).replace(/\-/g, '+').replace(/_/g, '/');
        const rawData = atob(base64);
        const outputArray = new Uint8Array(rawData.length);
        for (let i = 0; i < rawData.length; ++i) {
            outputArray[i] = rawData.charCodeAt(i);
        }
        return outputArray;
    }

    function ping () {
        ws.send('ping');
        timeout = setTimeout(() => {
            console.log('WebSocket connection closed. Please reload page.');
        }, 5000);
    }

    function pong () {
        clearTimeout(timeout);
    }

    function createSVG (icon) {
        const svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
        svg.classList.add('icon');
        svg.classList.add(`icon--${icon}`);
        const use = document.createElementNS('http://www.w3.org/2000/svg', 'use');
        use.setAttributeNS(
            'http://www.w3.org/1999/xlink',
            'href', 
            `/icons/sprite.svg#${icon}`)
        svg.appendChild(use);
        
        return svg;
    }

    function isValidHttpUrl(string) {
        let url;
        
        try {
            url = new URL(string);
        } catch (_) {
            return false;  
        }

        return url.protocol === "http:" || url.protocol === "https:";
    }

    function handleFeedClick (event) {
        document.getElementById('feedInput').value = '';
        ws.send(feedURL);
    }

    let prefix = '';
    const SUBDOMAIN = window.location.hostname.split(".")[0];
    if (SUBDOMAIN !== 'localhost') {
      prefix = SUBDOMAIN + '_';
    }
    const protocol = prefix + window.location.pathname.substring(1).replace(/\//g, '_').replace('@', '!');
    const ws = new WebSocket('wss://wtf.feed-dachau.de/ws/');
    //const ws = new WebSocket('ws://localhost:61716', protocol);

    let timeout;
    let feedURL;

    ws.onopen = () => setInterval(ping, 30000);

    ws.onmessage = message => {
        if (message.data === 'pong') {
            pong();
            return;
        }
        const feedArray = JSON.parse(message.data);
        const frag = document.createDocumentFragment();
        feedArray.forEach(feed => {
            // Date
            const date = document.createElement('span');
            const feedDate = new Date(feed.date);
            const formattedDate = new Intl.DateTimeFormat('de-DE').format(feedDate);
            date.textContent = formattedDate;

            // Time
            const time = document.createElement('span');
            const formattedTime = feedDate.toLocaleTimeString('de-De');
            time.textContent = formattedTime;

            // Source
            const source = document.createElement('span');
            source.className = 'text-truncate';
            let hostname;
            if (feed.link) {
                const url = new URL(feed.link);
                hostname = url.hostname;
            }
            else {
                hostname = 'de.feed.wtf';
            }
            if (hostname.startsWith('www.')) {
                hostname = hostname.replace('www.', '');
            }
            source.textContent = hostname;

            // Entry
            let entry;
            let details;
            if (feed.summary) {
                entry = document.createElement('summary');
                details = document.createElement('details');
                details.className = 'entry';
                details.textContent = feed.summary;
                details.appendChild(entry);
            }
            else {
                entry = document.createElement('div');
                entry.className = 'entry';
            }
            // Badge
            if (feedArray.length === 1) {
                const badge = document.createElement('span');
                badge.className = 'badge badge-secondary mr-2';
                badge.textContent = 'NEU';
                entry.appendChild(badge);
            }
            // Heading
            const linkHeading = document.createElement('h2');
            linkHeading.className = 'h6 d-inline';
            linkHeading.textContent = feed.title || 'Kein Feed gefunden. Bitte trage einen ein :)';
            entry.appendChild(linkHeading);

            // Social
            if (navigator.share) {
                const shareLink = document.createElement('a');
                shareLink.className = 'badge badge-secondary ml-2';
                shareLink.setAttribute('role', 'button');
                shareLink.setAttribute('aria-label', 'Beitrag teilen');
                shareLink.onclick = () => {
                    navigator.share({
                        title: feed.title,
                        url: feed.link
                    })
                    .then(() => console.log('Successful share'))
                    .catch((error) => console.log('Error sharing', error));
                }
                shareLink.appendChild(createSVG('share-2'));
                entry.appendChild(shareLink);
            }
            if (feed.link) {
                const externalLink = document.createElement('a');
                externalLink.className = 'badge badge-secondary ml-2';
                externalLink.href = feed.link;
                externalLink.rel = 'nofollow noopener';
                externalLink.setAttribute('aria-label', 'Seite aufrufen');
                externalLink.appendChild(createSVG('external-link'));
                entry.appendChild(externalLink);
                const facebookLink = document.createElement('a');
                facebookLink.className = 'badge badge-secondary ml-2';
                facebookLink.href = `https://www.facebook.com/sharer/sharer.php?u=${feed.link}`;
                facebookLink.rel = 'nofollow noopener';
                facebookLink.setAttribute('aria-label', 'Auf Facebook teilen');
                facebookLink.appendChild(createSVG('facebook'));
                entry.appendChild(facebookLink);
                const twitterLink = document.createElement('a');
                twitterLink.className = 'badge badge-secondary ml-2';
                twitterLink.href = `https://twitter.com/share?text=${feed.title}&url=${feed.link}`;
                twitterLink.rel = 'nofollow noopener';
                twitterLink.setAttribute('aria-label', 'Auf Twitter teilen');
                twitterLink.appendChild(createSVG('twitter'));
                entry.appendChild(twitterLink);
            }
            // Append all to frag
            frag.insertBefore(details || entry, frag.childNodes[0]);
            frag.insertBefore(source, frag.childNodes[0]);
            frag.insertBefore(time, frag.childNodes[0]);
            frag.insertBefore(date, frag.childNodes[0]);
        })
        window.requestAnimationFrame(() => {
            const feedbox = document.getElementById('feedbox');
            feedbox.insertBefore(frag, feedbox.childNodes[0]);
        });
    }
</script>

<section>
    <label for="feedInput">Eintragen</label>
    <input type="text" id="feedInput" name="feedInput" placeholder="Feed-URL" bind:value={feedURL}>
	<button id="feedButton" disabled={feedURL && isValidHttpUrl(feedURL) ? false : true} on:click={handleFeedClick}>Senden!</button>
</section>
<div id="feedbox"></div>
