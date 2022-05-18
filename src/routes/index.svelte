<script context="module" lang="ts">
	export const prerender = true;
</script>

<script lang="ts">
	type LDIF = {
		sn?: string;
		givenName?: string;
		mail?: string;
		title?: string;
		telephonenumber?: string;
		displayname?: string;
		mobile?: string;
		homePhone?: string;
		facsimileTelephoneNumber?: string;
		postalAddress?: string;
		l?: string;
		st?: string;
		postalCode?: string;
		homePostalAddress?: string;
		url?: string;
		description?: string;
	}
  let entries: LDIF[] = [];
  let output = "";
  let download: string;

  function convert_file(e: Event):void {
    let files:FileList = (<FileList>(<HTMLInputElement>e.target).files);
    const reader = new FileReader();
    reader.readAsText(files[0]);
    reader.onload = (e) => {
			if (!e.target) return
      const text = e.target.result || "";
			if (typeof text !== 'string') return;
      const data = text.split("\n\n");
      data.forEach((d) => {
        const e = d.split("\n");
        const n: LDIF = {};
        e.forEach((ee) => {
          if (ee.length) {
            const res = ee.match(/(.+):\s+(.+)/);
						if (res?.[1]) n[res[1]] = res[2] || "";
          }
        });
        entries = [...entries, n];
      });
      entries.forEach((e) => {
        output = output.concat(
`
BEGIN:VCARD
VERSION:3.0
N:${e.sn};${e.givenName};;;
FN:${e.displayname}
TITLE:${e.title}
NICKNAME:${e.givenName}
EMAIL;type=INTERNET:${e.mail}
TEL;type=WORK;type=VOICE:${e.telephonenumber}
tel;type=cell:${e.mobile}
tel;type=home:${e.homePhone}
tel;type=fax:${e.facsimileTelephoneNumber}
adr;type=work,postal,parcel:${e.postalAddress};${e.l};${ e.st };${e.postalCode};
adr;type=home,postal,parcel:${e.homePostalAddress};
item1.URL;type=pref:${e.url}
NOTE:${e.description}
END:VCARD

`
        );
      });
      download = `data:text/plain;charset=utf-8,${encodeURIComponent(output)}`;
    };
  };
</script>

<svelte:head>
	<title>LDIF nach VCARD3 Converter</title>
	<meta name="description" content="LDIF nach VCARD3 Converter" />
</svelte:head>

<section>
	<h1>
		Adressbuch von LDIF nach VCARD konvertieren
	</h1>
	<h2>
		Wählen Sie Ihr bereits aus Logineo exportiertes Adressbuch aus.
	</h2>
		<h2>Es werden keine Daten verschickt, die Konvertierung erfolgt im Browser.</h2>
	<br><input type="file" accept=".ldif" on:change={convert_file} />
</section>
<section>
	{#if download}
		<pre>{entries.length} Kontakte gefunden.</pre>
		<button>

			<a href={download} download="Adressen-VCARD.vcf"
			>Adressen im VCard-Format herunterladen</a
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
