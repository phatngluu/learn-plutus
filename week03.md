# Vesting
```haskell
đata VestingDatum = VestingDatum
{
	beneficiary :: PubKeyHash
, deadline :: POSIXTime
} deriving Show

PlutusTx.unstableMakeIsData ''VestingDatum

mkValidator :: VestingDatum -> () -> ScriptContext -> Bool
mkValidator dat () ctx = traceIfFalse "wrong beneficiary" signedByBeneficiary
												 traceIfFalse "deadline not reach" deadlineReached
where
	txInfo :: TxInfo
	txInfo = scriptContextTxInfo ctx
	signedByBeneficiary :: Bool
	signedByBeneficiary = txSignedBy txInfo $ beneficiary dat
	deadlineReached :: Bool
	deadlineReached = contains (from $ deadline dat) $ txInfoValidRange txInfo

data Vesting
instance Scripts.ValidatorTypes Vesting where
	type instance DatumType Vesting = VestingDatum
	type instance RedeemerType Vesting = ()

typedValidator :: Scripts.TypedValidator Vesting
typedValidator = Scripts.mkTypedValidator @Vesting
	$$(PlutusTx.compile [|| mkValidator ||])
	$$(PlutusTx.compile [|| wrap ||])
where
	wrap = Scripts.wrapValidator @VestingDatum @()

-- offchain
type VestingSchema = 
	.\/ Endpoint "give" GiveParams -- call with params
	.\/ Endpoint "grab" () -- call without params

data GiveParams = GiveParams
	{ gpBeneficiary :: !PubKeyHash
	, gpDeadline :: !POSIXTime
	, gpAmount :: !Integer
} deriving (Generic, ToJSON, FromJSON, ToSchema)

give :: AsContractError e => GiveParams -> Contract w s e ()
give gp = do
	let dat = VestingDatum
							{ beneficiary = gpBeneficiary gp
							, deadline    = gpDeadline gp
							}
			tx = mustPayToTheScript dat $ Ada.lovelaceValueOf $ gpAmount gp
```

# API Notes
```haskell
-- https://alpha.marlowe.iohkdev.io/doc/haddock/plutus-ledger-api/html/Plutus-V1-Ledger-Contexts.html#t:ScriptContext
ScriptContext
	scriptContextTxInfo :: TxInfo
	scriptContextPurpose :: ScriptPurpose

-- https://alpha.marlowe.iohkdev.io/doc/haddock/plutus-ledger-api/html/Plutus-V1-Ledger-Contexts.html#v:txSignedBy
txSignedBy :: TxInfo -> PubKeyHash -> Bool

-- https://alpha.marlowe.iohkdev.io/doc/haddock/plutus-ledger-api/html/Plutus-V1-Ledger-Interval.html#v:contains
contains :: Ord a => Interval a -> Interval a -> Bool

-- https://alpha.marlowe.iohkdev.io/doc/haddock/plutus-ledger-api/html/Plutus-V1-Ledger-Contexts.html#t:TxInfo
TxInfo
	txInfoValidRange :: POSIXTimeRange

-- offchain
-- https://alpha.marlowe.iohkdev.io/doc/haddock/plutus-ledger/html/Ledger-Constraints-TxConstraints.html
mustPayToTheScript :: forall i o. ToData o => o -> Value -> TxConstraints i o
-- Lock the value with a script
mustPayToOtherScript :: forall i o. ValidatorHash -> Datum -> Value -> TxConstraints i o
-- Lock the value with a public key
-- https://alpha.marlowe.iohkdev.io/doc/haddock/plutus-ledger-api/html/Plutus-V1-Ledger-Ada.html
lovelaceValueOf :: Integer -> Value

```