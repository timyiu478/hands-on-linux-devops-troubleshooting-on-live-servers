## "Woluwe": Too many images

### Problem

Description: A pipeline created a lot of Docker images locally for a web app. All these images except for one contain a typo introduced by a developer: there's an incorrect image instruction to pipe "HelloWorld" to "index.htmlz" instead of using the correct "index.html"
Find which image doesn't have the typo (and uses the correct "index.html"), tag this correct image as "prod" (rather than fixing the current prod image) and then deploy it with docker run -d --name prod -p 3000:3000 prod so it responds correctly to HTTP requests on port :3000 instead of "404 Not Found".

Root (sudo) Access: True

Test: curl http://localhost:3000 should respond with HelloWorld;529

### Solution

#### 1. write a script to find and deploy it

Co-pilot: Grok 4.1

Script:

```bash
#!/bin/bash
set -euo pipefail

echo "Scanning all local Docker images..."

GOOD_ID=$(docker images --quiet --no-trunc | sort -u | while read -r img_id; do
    # Skip empty lines
    [ -z "$img_id" ] && continue

    HISTORY=$(docker history --no-trunc "$img_id" 2>/dev/null || true)

    # Case 1: has the typo → skip
    if echo "$HISTORY" | grep -q 'index\.htmlz'; then
        echo "BAD  $img_id  (contains index.htmlz)" >&2
        continue
    fi

    # Case 2: does NOT have the typo, but must contain the correct file
    if echo "$HISTORY" | grep -q 'index\.html'; then
        echo "GOOD $img_id  (uses correct index.html)" >&2
        echo "$img_id"
    else
        echo "SKIP $img_id  (no index.html at all)" >&2
    fi
done | head -1)   # only one good image exists

if [ -z "$GOOD_ID" ]; then
    echo "ERROR: No correct image found!"
    exit 1
fi

echo "Correct image found: $GOOD_ID"

# Tag and deploy
docker tag "$GOOD_ID" prod
docker rm -f prod 2>/dev/null || true
docker run -d --name prod -p 3000:3000 prod

echo "Deployed correct image $GOOD_ID as 'prod'"
echo "App is now running correctly at http://localhost:3000"
curl -s http://localhost:3000 > /dev/null && echo "Health check: OK" || echo "Health check: FAILED"
```

#### 2. run the script

```
admin@i-0bb15e2e2e010d1f8:~$ ./find.sh 
Scanning all local Docker images...
BAD  sha256:00f8cadc6bf47655fec3cf913decd151fec9cdc67664232b0d9090534919edbe  (contains index.htmlz)
BAD  sha256:047e0e23e348b2bc9b309b6b695501736752e5d157f8a386ded50c0250c0984e  (contains index.htmlz)
BAD  sha256:04ec08f869cd446febfe6137cdf4ff07475f66472fecc9961a283eecb6313e46  (contains index.htmlz)
BAD  sha256:0ac49b75b0ac6592857435dbbf37b211d4bf47885a3bf3c70159c1275c8ee472  (contains index.htmlz)
BAD  sha256:0ac5927ec39d48a0980ef5c5727df1ce298a57799d820f8d11d67a340d0dbe22  (contains index.htmlz)
BAD  sha256:0b03e9dc5bb891216cedd53462f445c15d878a8e9cde2c6db0fa9429355d929c  (contains index.htmlz)
BAD  sha256:0d253c6d4a6eaee571dfce99d1a8344e0d917f3961bcb88c20a45634ff1bbdf2  (contains index.htmlz)
BAD  sha256:0fae564cf8a39bf3623996d23435537614218c53d3431ffb3b6ef92e1926492e  (contains index.htmlz)
BAD  sha256:116e2b9dcf8908b3686238d168343c2f2a4b9e6720ac6cdc0075e8c837f834a3  (contains index.htmlz)
BAD  sha256:1231eb0abbe4f2d9cbf6a708dfca0af6c9544ec3d2ee484c65b77bf3cf1919a9  (contains index.htmlz)
BAD  sha256:1f73440c64c1a35161dad5e5d3dd03b252bcd32a2b94bf9cfa5b458ee8391cd8  (contains index.htmlz)
BAD  sha256:24b4c86be06715113871e0737d60dda885da635971a468d1e8c0a8de40a7ece4  (contains index.htmlz)
BAD  sha256:2d2c9663c259e2e3768f4bc25c7f585f8a7dc93f1e34614b6a53ef05bf8eafd0  (contains index.htmlz)
BAD  sha256:2e3c60654baec637df2e70f5b50bf30c2fc7c892859f41a1a6518d5a7890eb74  (contains index.htmlz)
BAD  sha256:30c566ccb2b8e46e3f082bf9def7c4a8413f6ffecad823beacc99624e1c72ab9  (contains index.htmlz)
BAD  sha256:3233cb6d53273fdc70924e9011e6503a85c27fa2fa742cfd75acab4ff33bbb6f  (contains index.htmlz)
BAD  sha256:36f635da0eba98946497eb5e29e0309a6a3f8a14ab26f6febc2ec4ba48651819  (contains index.htmlz)
BAD  sha256:37f1b9840b69e32d8bbfaf45ea6688cdd94a0cc890a28b5241dbe14690289336  (contains index.htmlz)
BAD  sha256:3877cd7c3db6303c0a7476661d12b2513b88c955c7c9d2ad2465bde541704a10  (contains index.htmlz)
BAD  sha256:3d028dce31ac1a22f58a30fc05e1f6a34e9139038aed0195cc4bcb23f9e8a65a  (contains index.htmlz)
BAD  sha256:3e3f4d9e006f257bd7e33d06e0f18ff26a3fa4e7282bb6c3b2d3216b0b2b5363  (contains index.htmlz)
BAD  sha256:3ec7b39680930dade9db9e637671b9165f76c8cd484074235587994fab5ae055  (contains index.htmlz)
GOOD sha256:3f8befa65f011134767c89fa24709ffa01ef81b055b894c8a0b0f43fe37dcd34  (uses correct index.html)
BAD  sha256:4181cb5d8a972945cb3bdd5cfdbb983577ce9a99324baf4d697c8c78f1cf819f  (contains index.htmlz)
BAD  sha256:42c2727f1b971c6adfe175c15b47a884cd1e9b2bfed81ac8b56df319d070685c  (contains index.htmlz)
BAD  sha256:42eac5f5c3464a6ea15565cc9981c6d534e5b3e56c4512f4a024eb3b2c4ac05c  (contains index.htmlz)
BAD  sha256:4687c099dee25b0754026d2135f160e48ab9a04395d6af9cb11e8896ca917840  (contains index.htmlz)
BAD  sha256:48272323e6db9dd4e580dbb29565579fcc6481eddc95a81303502ebbcd1f0250  (contains index.htmlz)
BAD  sha256:4b0bc57647dabd353cd476fd3ed4a3365db0cfb8b547f58b2c7ae150a969a8a0  (contains index.htmlz)
BAD  sha256:4bbc151d06f6bcf69637b966d51d05a2eebe14baa2dde95c78365b175d50eb07  (contains index.htmlz)
BAD  sha256:4c75e0bdb5fa73918515dd509401cc8bebfdd29eb4db34120e346eb8a0847b89  (contains index.htmlz)
BAD  sha256:52eab2232d8ea4c134bbb5020e1d1d4bf8d213a9b974c4aed47030ab3bfab23d  (contains index.htmlz)
BAD  sha256:54ee3223d8c7eba8b5602931ed5726ee07427097a49bf7804f7366b6c382ea61  (contains index.htmlz)
BAD  sha256:56d411f823aa693fd905dd6bd3b8ffeddbeb0d62fe0b6e84840a5427776ab8c8  (contains index.htmlz)
BAD  sha256:56da80dcd1adab1689faabf8823cdbfff5eca8375cd2d80f62dcba70750457e1  (contains index.htmlz)
BAD  sha256:5a9a371664b6a74f8555f68adecec87ce1b659799a9b0d6fa8a63990cf3bbc47  (contains index.htmlz)
BAD  sha256:5c21716498a10e98ef141f50b79e91ce0bf640de94e8826f1353c1d069bd3f22  (contains index.htmlz)
BAD  sha256:5e342c84869824f08122321b0016c87efb37b78cbe20bef7a1d7120d28ea92b1  (contains index.htmlz)
BAD  sha256:5e776202c489e39053c231f499b1bbeed8ef88768bb699d4ff2319a53ff36710  (contains index.htmlz)
BAD  sha256:5f90739e8cfaed095e905196cf1d67c430e8cd74478d5003eb82ae164f9a84de  (contains index.htmlz)
BAD  sha256:5fa147e5c8c9fd6c63988508cbf8567e255ab0fe1302f962bbea44f4f5c50c09  (contains index.htmlz)
BAD  sha256:62c384637516f883424dbe54cf21f99840963a60bddd7559f9a1c54576eb6421  (contains index.htmlz)
BAD  sha256:6a2951b3311f51eb2e2fafffef1968329761e5116f43e362d352063c3c13d35f  (contains index.htmlz)
BAD  sha256:6ac68f5f121d4da21027208a5441848eda52cb47a050ff4f90f9dd2604d4b882  (contains index.htmlz)
BAD  sha256:6ce8cc718ad1e7be13115bcd0a8f1e0d12bfa9dbb2ac0bfd7a5e4c99645595b3  (contains index.htmlz)
BAD  sha256:6e1af00cdf5cc1842204ba7403e86263cfaf81aeb6fbc11136885b611145c9bc  (contains index.htmlz)
BAD  sha256:6ebd212e6034bdbe0364a88a13a3ef08680554a74b3a1b72650c7ada24420063  (contains index.htmlz)
BAD  sha256:707809e03f81dfea960be78b32a41659752fe7e04036b27e15cd0db69bdd582d  (contains index.htmlz)
BAD  sha256:70daedc3fbdd5c97e964a13e5cfb51a4f3050efcd7e6703a4a036b3ddf556e55  (contains index.htmlz)
BAD  sha256:78c8131f5a74773f6887673e190586d6f0255ae096220e037968cded7d9e5dab  (contains index.htmlz)
BAD  sha256:7ef77e0bb0728060a2a7fc86a72fa5479127cf48d8b6179cf7d59661b0d86968  (contains index.htmlz)
BAD  sha256:8114437d5aa3fc69ebbe3c3f180d357aa917e09801b220c08d3d75591d1bc2c3  (contains index.htmlz)
BAD  sha256:82058d272342c81521276f295d8774c6bfedfd2499eeb65f96984a6e395a60d2  (contains index.htmlz)
BAD  sha256:8236dee69cea38018d2f463be771905a5f4c22564355f1547a0e4d65628d244a  (contains index.htmlz)
BAD  sha256:841990e44da7ef388d271f40d370831da4961cfe0b02dc3db1aa5259bb47044b  (contains index.htmlz)
BAD  sha256:84632a29cde3b5b1c600fc2b5d6958246486ab01fdfbc7d12f3cf219ba2d5dcd  (contains index.htmlz)
BAD  sha256:92842993e01b0c45b849cc124d7f012ffa3ec1aada14936c1fbfbface2783319  (contains index.htmlz)
BAD  sha256:948b71cdb062310e5dfc2899663a85307ff40da48f3a8991a9d4eafbf809021c  (contains index.htmlz)
BAD  sha256:9b8e465f3b74535af1da4fb5b558b3f5690fd63bff0e94e80302b3dc1f42e944  (contains index.htmlz)
BAD  sha256:9fc37b823926aa343716f2f3f7a8c993f81a5cc81f2bd1d6a6244164494f9494  (contains index.htmlz)
BAD  sha256:a4e77f0824f716e72dd9103d4d4bcc6f01bfd4995db65bcb194d0da8d4029b1b  (contains index.htmlz)
BAD  sha256:a651624414d394ad92d16e9302d414f7011c165d0fda8bfbee6add6eb99ab16a  (contains index.htmlz)
BAD  sha256:a7adbfa6084ad371ec0fd80747fdd1a3895a5bf865a26839a745c83c4c58481f  (contains index.htmlz)
BAD  sha256:ac9f5b7a4ed6768ccb3f1091ae957d2dccf33ea211d2d0fd17df9e40e4f3fc1c  (contains index.htmlz)
BAD  sha256:ad9c706df5da9a192757b1963e8b85c99d52dab693f174e32b13ced798ffea4b  (contains index.htmlz)
BAD  sha256:ae60c7a9a45377380eb83d95df621e0a3e7671714c959a456acea078b4f2dc31  (contains index.htmlz)
BAD  sha256:ae7b8b0f7a9a1f0a635ccdf4027fcb9b82856c8713923a4d40e8641c2df180e4  (contains index.htmlz)
BAD  sha256:b0f2967fe08be2e5018477debcf22c6653abb9e67b1b7d54fddca1b9c7f340d7  (contains index.htmlz)
BAD  sha256:b27420d8b3a1e71642ff02325ef4329b3f74ae0479d26274a8b460b640463276  (contains index.htmlz)
BAD  sha256:b322518eb13bbe71c3a9455e8c698b5b1a51e7dcdfc2064e376fdcb822b1bd97  (contains index.htmlz)
BAD  sha256:b68ff4a7834092df5b757f601ce6f1cf53c7ac2efcda4dbb1c99aa2cdcf9160f  (contains index.htmlz)
BAD  sha256:b99e69d1ce8f0a77853713ffd95caf12025e69c1c0bcc0b95bd892117600d203  (contains index.htmlz)
BAD  sha256:be8eb2a7030445454e1cea6fb25950631a55cf0e7c5294a21543ecc6902b1c40  (contains index.htmlz)
BAD  sha256:c063aea926b76daaa8426ad7b8707bbf4f976c306052deb56d24873678800410  (contains index.htmlz)
BAD  sha256:c272870c91a4f3ace018072046fd623d973cdb0666450f0992aa65fbc32b9910  (contains index.htmlz)
BAD  sha256:c38d399830684f8647dbdc5835ac4bed6a8dc986f8876537f0fabb3f32bfd0f0  (contains index.htmlz)
BAD  sha256:c66f5eae05e040d270e895d0e3dfc1e9dcedda7497293ee57903d5f9d080a9f4  (contains index.htmlz)
BAD  sha256:c6d17ba8477e5132112f0d432e8b7181031a799ca029b5943707c85e0cdc8054  (contains index.htmlz)
BAD  sha256:cc5260c6392e527621b90393528d7fc085f262201c08ae06e3be933cb2fe8197  (contains index.htmlz)
BAD  sha256:ce83398fc47962be46980e3b7ac26d73517a90c3393c5d0f5d353d349590dc7c  (contains index.htmlz)
BAD  sha256:d27184088f2c3212c36cbb53265393407fbeade3e8348355c59007acd4f6d1af  (contains index.htmlz)
BAD  sha256:d2b9ff9018c22bda8e2bf8bd2941c6ae82b843de687fcb23f947e9a51d47ad1d  (contains index.htmlz)
BAD  sha256:d5c7640d17b0cbdc24d469be4e5ce555026672e8e31ddbbc28fcc238f1e3ae54  (contains index.htmlz)
BAD  sha256:d72b953574fd05e46eafcf08f2867ad8aabeba072e00cd08c38934580da09039  (contains index.htmlz)
SKIP sha256:dd15126afe8d37c7a5d14e6da501763ab4e260e33edf2f70e0fb9e2ee3b80f87  (no index.html at all)
BAD  sha256:e105efb912780fc8ab83a0a983a79133bc8bf8773036f92dfdd7a3439546227f  (contains index.htmlz)
BAD  sha256:e1cabeb3333eb6b1380bbb9fa2c716e56eea5a69e7d3c7d46f7d11620b81d5a5  (contains index.htmlz)
BAD  sha256:e1e9bb45dff29985e4cdf25851b13220502fe79e3b4c9242a4ebbbc775836ebf  (contains index.htmlz)
BAD  sha256:e2d62f718663fee1b111d93dcb43dec098bb578e8e4e8aaa1c6cf13c3c13b681  (contains index.htmlz)
BAD  sha256:e47fb961b1b834e7cfff060b9a8437e3eeceded8fd39070c831c167e9a4ed385  (contains index.htmlz)
BAD  sha256:e8ce0cae6737aed19ee59c63144ac953a6455d1c777fecc56980981d6bb0dd57  (contains index.htmlz)
BAD  sha256:e991a67a5388f0b5ad2fa19f20d4c1cb7ead5955d81217ba812dbf3e98301848  (contains index.htmlz)
BAD  sha256:ece8d83e5b5695e3643da8026ba9fe8a917be915a85b1b2023944cea6b02e259  (contains index.htmlz)
BAD  sha256:ede6fd52c6063f9435fd905121963adfc07fac0b4ebc508ee69c1a34b07b5883  (contains index.htmlz)
BAD  sha256:ee5d1f847306e854d6212b0e72c0c3abf29b244dd97c36456987c1bd065e6996  (contains index.htmlz)
BAD  sha256:ef28344e373e538276cd1834b68ef796dd95fe6230c44631d222bdde7872b3fe  (contains index.htmlz)
BAD  sha256:f475138f78df9b04e0ec53114e8928366a6f2cee953783a991a3a74ad5744f56  (contains index.htmlz)
BAD  sha256:f5b8d0e34b9aa9f8a990c15f34e8519e1b21217aad5603d321cee932984bc403  (contains index.htmlz)
BAD  sha256:f8c4978a4144415c08cc34e944cc0c8010be7d10e33efc62215f40a20444ed8b  (contains index.htmlz)
BAD  sha256:f8f54378cd09a48ef5a4755deeb705b7c87d7e022f8f5b4c3ea4442b3eca7f4f  (contains index.htmlz)
BAD  sha256:fafbbe0f36823aa04f83d1846a32ae9cbffd22a503c4f2df43d530e3a026a536  (contains index.htmlz)
BAD  sha256:fbe021c44bdc6200bb85b55484ca9de2413e8863ca9202811132160e43a97017  (contains index.htmlz)
Correct image found: sha256:3f8befa65f011134767c89fa24709ffa01ef81b055b894c8a0b0f43fe37dcd34
prod
1bbd6e41d34100f2ca02cc48af770c798ed70c2ab8b45e7d790a2ca2ca53e5b6
Deployed correct image sha256:3f8befa65f011134767c89fa24709ffa01ef81b055b894c8a0b0f43fe37dcd34 as 'prod'
App is now running correctly at http://localhost:3000
Health check: OK
```
