<script context="module" lang="ts">
	export const prerender = true;
</script>

<script lang="ts">
	type LDIF = {
		objectClass?: string;
		objectclass?: string;
		sn?: string;
		givenname?: string;
		mail?: string | Array<string>;
		title?: string;
		telephonenumber?: string | Array<string>;
		displayname?: string;
		mobile?: string | Array<string>;
		homePhone?: string | Array<string>;
		facsimileTelephoneNumber?: string | Array<string>;
		postalAddress?: string;
		l?: string;
		st?: string;
		postalCode?: string;
		homePostalAddress?: string;
		url?: string | Array<string>;
		description?: string;
		cn?: string;
		member?: string | Array<string>;
	};
	let entries: LDIF[] = [];
	let output = '';
	let download: string;

	function convert_file(e: Event): void {
		let files: FileList = <FileList>(<HTMLInputElement>e.target).files;
		const reader = new FileReader();
		reader.readAsText(files[0]);
		reader.onload = (e) => {
			if (!e.target) return;
			const text = e.target.result || '';
			if (typeof text !== 'string') return;
			const data = text.split('\n\n');
			data.forEach((d) => {
				const e = d.split('\n');
				const n: LDIF = {};
				e.forEach((ee) => {
					if (ee.length) {
						const res = ee.match(/(.+):\s+(.+)/);
						if (res?.[1]) {
							if (!n[res[1]]) n[res[1]] = res[2];
							else {
								n[res[1]] =
									n[res[1]] instanceof Array ? [...n[res[1]], res[2]] : [n[res[1]], res[2]];
							}
						}
					}
				});
				entries = [...entries, n];
			});
			entries.forEach((e) => {
				if (e.objectClass?.includes('inetOrgPerson')) {
					output += `
BEGIN:VCARD
VERSION:3.0
N:${e.sn || ''};${e.givenname || ''}
FN:${e.displayname || ''}
TITLE:${e.title || ''}
NICKNAME:${e.givenname || ''}\n`;
					if (e.mail instanceof Array) {
						e.mail.forEach((mail) => (output += `EMAIL;type=INTERNET:${mail || ''}\n`));
					} else output += `EMAIL;type=INTERNET:${e.mail || ''}\n`;

					if (e.telephonenumber instanceof Array) {
						e.telephonenumber.forEach(
							(telephonenumber) => (output += `TEL;type=WORK;type=VOICE:${telephonenumber || ''}\n`)
						);
					} else output += `TEL;type=WORK;type=VOICE:${e.telephonenumber || ''}\n`;

					if (e.mobile instanceof Array) {
						e.mobile.forEach((mobile) => (output += `tel;type=cell:${mobile || ''}\n`));
					} else output += `tel;type=cell:${e.mobile || ''}\n`;

					if (e.homePhone instanceof Array) {
						e.homePhone.forEach((homePhone) => (output += `tel;type=home:${homePhone || ''}\n`));
					} else output += `tel;type=home:${e.homePhone || ''}\n`;

					if (e.facsimileTelephoneNumber instanceof Array) {
						e.facsimileTelephoneNumber.forEach(
							(facsimileTelephoneNumber) =>
								(output += `tel;type=fax:${facsimileTelephoneNumber || ''}\n`)
						);
					} else output += `tel;type=fax:${e.facsimileTelephoneNumber || ''}\n`;

					if (e.url instanceof Array) {
						e.url.forEach((url) => (output += `item1.URL;type=pref:${url || ''}\n`));
					} else output += `item1.URL;type=pref:${e.url || ''}\n`;

					output += `adr;type=work,postal,parcel:${e.postalAddress || ''};${e.l || ''};${
						e.st || ''
					};${e.postalCode || ''}
adr;type=home,postal,parcel:${e.homePostalAddress || ''}
NOTE:${e.description || ''}
END:VCARD

`;
				} else if (e.objectclass?.includes('groupOfNames')) {
					if (!e.cn?.startsWith(':')) {
						output += `BEGIN:VCARD
VERSION:3.0
FN:${e.cn}
KIND:group\n`;
						if (e.member instanceof Array) {
							e.member.forEach((member) => {
								const res = member.match(/cn=(.+),mail=(.*)/);
								console.log(member, res);
								output += `MEMBER;X-OX-FN=${res?.[1] || ''}:mailto:${res?.[2] || ''}\n`;
							});
						} else {
							const res = e.member?.match(/cn=(.+),mail=(.*)/);
							output += `MEMBER;X-OX-FN=${res?.[1] || ''}:mailto:${res?.[2] || ''}\n`;
						}
						output += 'END:VCARD\n\n';
					}
				}
			});
			download = `data:text/plain;charset=utf-8,${encodeURIComponent(output)}`;
		};
	}
</script>

<svelte:head>
	<title>LDIF nach VCARD3 Converter</title>
	<meta name="description" content="LDIF nach VCARD3 Converter" />
</svelte:head>

<section>
	<h1>Adressbuch von LDIF nach VCARD konvertieren</h1>
	<h2>Wählen Sie Ihr bereits aus Logineo exportiertes Adressbuch aus.</h2>
	<h2>Es werden keine Daten verschickt, die Konvertierung erfolgt im Browser.</h2>
	<br /><input type="file" accept=".ldif" on:change={convert_file} />
</section>
<section>
	{#if download}
		<pre>{entries.length} Kontakte gefunden.</pre>
		<button>
			<a href={download} download="Adressen-VCARD.vcf"
				>Adressen und Verteilerlisten im VCard-Format herunterladen</a
			>
		</button>
	{/if}
</section>

<style>
	section {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		flex: 1;
		margin: 2em;
	}

	h1 {
		width: 100%;
	}
</style>
