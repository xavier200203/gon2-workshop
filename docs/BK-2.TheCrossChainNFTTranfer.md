# 2. The cross-chain NFT tranfer
## Overview

This part mainly introduce the cross-chain of [ICS721 SDK chain] and [ICS721 Wasm chain] by implementing two cross-chain Demos.


### 2.1 ICS721 SDK Chain Cross-Chain Demo (Uptick <-> IRIS)


#### 2.1.1 issue NFT

```sh
uptickd tx collection issue uptickCollection --from=uptickKey --name=uptickCollectionName \
--symbol="uptick collection symbol" --mint-restricted=false --update-restricted=false \
--schema="uptick collection schema" --description="uptick collection description" \
--uri="uptick collection uri" --uri-hash="uri-hash" --data="{\"data\":\"uptickData\"}" \
--fees=100auptick -b block -y

uptickd query collection denom uptickCollection
```

#### 2.1.2 Mint NFT
```sh
uptickd tx collection mint uptickCollection id001 \
--uri="uptick mint uri" --uri-hash="uri-mint-hash" --name="uptick token name" \
--recipient="uptick1dzla4hhzwp7xz0qgcnq38c8rth58v7jdyg80xc"  \
--chain-id=uptick_7000-2 --fees=100auptick --from=uptickKey -b block -y

uptickd query collection token uptickCollection id001
```

#### 2.1.3 Uptick -> IRIS

```sh
uptickd tx nft-transfer transfer \
nft-transfer channel-3 \
iaa1pzl8eqp4t7f8c3h6e43pdftcx2qvgn5dltr47z uptickCollection id001 \
--chain-id=uptick_7000-2 --fees=100auptick --from=uptickKey -b block -y

#check in uptick chain
uptickd query collection token uptickCollection id001

#check in iris chain
iris query nft owner iaa1pzl8eqp4t7f8c3h6e43pdftcx2qvgn5dltr47z

#check class info 
iris  query nft-transfer class-trace 3CAE06D2AF60F79CF3E918BC71EDCBB980D273861915EFB43087C5B42B288C56
```

#### 2.1.4 IRIS- > Uptick
```sh
iris tx nft-transfer transfer \
nft-transfer channel-17 \
uptick1dzla4hhzwp7xz0qgcnq38c8rth58v7jdyg80xc ibc/3CAE06D2AF60F79CF3E918BC71EDCBB980D273861915EFB43087C5B42B288C56 id001 \
--from=iaa1pzl8eqp4t7f8c3h6e43pdftcx2qvgn5dltr47z --chain-id=gon-irishub-1 --fees=10000uiris -b block -y

#check in iris chain
iris query nft owner iaa1pzl8eqp4t7f8c3h6e43pdftcx2qvgn5dltr47z

#check in uptick chain
uptickd query collection token uptickCollection id001

```


### 2.2 ICS721 Wasm Chain Cross-Chain Demo (Uptick <-> Stargaze)


#### 2.2.1 Uptick -> Stargaze
```sh
uptickd tx nft-transfer transfer \
nft-transfer channel-6 \
stars18ajmlej8nkyl9qtwhrc6kynwr6tuuf7xmtsfxh uptickCollection id001 \
--chain-id=uptick_7000-2 --fees=100auptick --from=uptickKey -b block -y

#check in uptick chain
uptickd query collection token uptickCollection id001

#get contract address from stargazes chain
starsd query wasm contract-state smart stars1ve46fjrhcrum94c7d8yc2wsdz8cpuw73503e8qn9r44spr6dw0lsvmvtqh '{"nft_contract":{"class_id":"wasm.stars1ve46fjrhcrum94c7d8yc2wsdz8cpuw73503e8qn9r44spr6dw0lsvmvtqh/channel-203/uptickCollection"}}'

#check in stargazes chain
starsd query wasm contract-state smart stars1tzj53s4zvawvftylhjtdu58ya2c3p36thc3fx42ynk7n7exgsntq3j6dr2 \
'{"all_nft_info":{"token_id": "id001"}}'


```

#### 2.2.2 Stargaze- > Uptick

+ base64 encode tranfer infomation

```sh
#base64 encode tranfer info
echo "{ \"receiver\": \"uptick1dzla4hhzwp7xz0qgcnq38c8rth58v7jdyg80xc\", \"channel_id\": \"channel-203\", \"timeout\": { \"block\": { \"revision\": 2, \"height\": 3999999 } } }" | base64
#eyAicmVjZWl2ZXIiOiAidXB0aWNrMWR6bGE0aGh6d3A3eHowcWdjbnEzOGM4cnRoNTh2N2pkeWc4MHhjIiwgImNoYW5uZWxfaWQiOiAiY2hhbm5lbC0yMDMiLCAidGltZW91dCI6IHsgImJsb2NrIjogeyAicmV2aXNpb24iOiAyLCAiaGVpZ2h0IjogMzk5OTk5OSB9IH0gfQo=

starsd tx wasm execute stars1tzj53s4zvawvftylhjtdu58ya2c3p36thc3fx42ynk7n7exgsntq3j6dr2 \
'{"send_nft":{"contract":"stars1ve46fjrhcrum94c7d8yc2wsdz8cpuw73503e8qn9r44spr6dw0lsvmvtqh","token_id":"id001","msg":"eyAicmVjZWl2ZXIiOiAidXB0aWNrMWR6bGE0aGh6d3A3eHowcWdjbnEzOGM4cnRoNTh2N2pkeWc4MHhjIiwgImNoYW5uZWxfaWQiOiAiY2hhbm5lbC0yMDMiLCAidGltZW91dCI6IHsgImJsb2NrIjogeyAicmV2aXNpb24iOiAyLCAiaGVpZ2h0IjogMzk5OTk5OSB9IH0gfQo="}}' \
--gas-prices 0.1ustars --gas auto --gas-adjustment 1.3 -b block --from starsKey --yes

#check in uptick chain
uptickd query collection token uptickCollection id001

#check in Stargaze chain
starsd query wasm contract-state smart stars1tzj53s4zvawvftylhjtdu58ya2c3p36thc3fx42ynk7n7exgsntq3j6dr2 \
'{"all_nft_info":{"token_id": "id001"}}'

```





