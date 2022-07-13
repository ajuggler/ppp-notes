---
label: Lecture 4
icon: file
author:
  name: Antonio Hernandez
  email: contacto@antoniohernandez.mx
order: -7
---

# Emulator Trace and Contract Monads

[Lecture Videos :icon-link-external:](https://www.youtube.com/playlist?list=PLNEK_Ejlx3x230-g-U02issX5BiWAgmSi)

Topics covered:

1. Introduction
2. Monads
3. The EmulatorTrace Monad
4. The Contract Monad
5. Homework & Summary

## Running `EmulatorTrace`

Note that `EmulatorConfig` implements an instance of the `Default` type class:

    > import Plutus.Trace.Emulator
    > import Data.Default
    > 
    > :i EmulatorConfig
    type EmulatorConfig :: *
    data EmulatorConfig
      = EmulatorConfig {_initialChainState :: Wallet.Emulator.Stream.InitialChainState,
    		    _slotConfig :: Ledger.TimeSlot.SlotConfig,
    		    _feeConfig :: Ledger.Fee.FeeConfig}
    	-- Defined in ‘Wallet.Emulator.Stream’
    instance Eq EmulatorConfig -- Defined in ‘Wallet.Emulator.Stream’
    instance Show EmulatorConfig -- Defined in ‘Wallet.Emulator.Stream’
    instance Default EmulatorConfig
      -- Defined in ‘Wallet.Emulator.Stream’

so that we can call *defaults* of `EmulatorTrace` as follows:

    > def :: EmulatorConfig

with output:

    EmulatorConfig {_initialChainState = Left (fromList [(Wallet 1bc5f27d7b4e20083977418e839e429d00cc87f3,Value (Map [(,Map [("",100000000)])])),(Wallet 3a4778247ad35117d7c3150d194da389f3148f4a,Value (Map [(,Map [("",100000000)])])),(Wallet 4e76ce6b3f12c6cc5a6a2545f6770d2bcb360648,Value (Map [(,Map [("",100000000)])])),(Wallet 5f5a4f5f465580a5500b9a9cede7f4e014a37ea8,Value (Map [(,Map [("",100000000)])])),(Wallet 7ce812d7a4770bbf58004067665c3a48f28ddd58,Value (Map [(,Map [("",100000000)])])),(Wallet 872cb83b5ee40eb23bfdab1772660c822a48d491,Value (Map [(,Map [("",100000000)])])),(Wallet bdf8dbca0cadeb365480c6ec29ec746a2b85274f,Value (Map [(,Map [("",100000000)])])),(Wallet c19599f22890ced15c6a87222302109e83b78bdf,Value (Map [(,Map [("",100000000)])])),(Wallet c30efb78b4e272685c1f9f0c93787fd4b6743154,Value (Map [(,Map [("",100000000)])])),(Wallet d3eddd0d37989746b029a0e050386bc425363901,Value (Map [(,Map [("",100000000)])]))]), _slotConfig = SlotConfig {scSlotLength = 1000, scSlotZeroTime = POSIXTime {getPOSIXTime = 1596059091000}}, _feeConfig = FeeConfig {fcConstantFee = Lovelace {getLovelace = 10}, fcScriptsFeeFactor = 1.0}}

which means that by default we have 10 wallets and other defaults having to do with 'SlotConfig', time and fees.

Since the `EmulatorTrace` is a monad, the simplest trace that we can run is tha one that returns *unit*.  We can try it using `runEmulatorTrace`:

    > :t runEmulatorTrace
    runEmulatorTrace
      :: EmulatorConfig
         -> EmulatorTrace ()
         -> ([Wallet.Emulator.MultiAgent.EmulatorEvent], Maybe EmulatorErr,
    	 Wallet.Emulator.MultiAgent.EmulatorState)

so we have to provide an `EmulatorConfig` and a contract.  We provide defaults and the simplest trace consisting of `return ()` :

    > writeFile "out.txt" $ show $ runEmulatorTrace def $ return ()

but this gives an unwieldly large result:

    > :! ls -lah out.txt
    -rw-r--r-- 1 antonio staff 439K Jul 12 18:51 out.txt

so even an emulator that doesn't do anything yields a large number of emulation events.  Therefore `runEmulatorTrace` is not a feasible way to try out contracts.

Instead we will use `runEmulatorTraceIO` :

    > :t runEmulatorTraceIO
    runEmulatorTraceIO :: EmulatorTrace () -> IO ()

which does not need to be provided an emulator configuration (it takes the defaults) and maps the `EmulatorTrace` monad to the `IO` monad.

    > runEmulatorTraceIO $ return ()
    Slot 00000: TxnValidate 98d5fbcefe21113b3f0390c1441e075b8a870cc5a8fa2a56dcde1d8247e41715
    Slot 00000: SlotAdd Slot 1
    Slot 00001: W1bc5f27: InsertionSuccess: New tip is Tip(slot= Slot 1, blockId= BlockId(1033bd6bfb9d90108db08880ad32a58980ae8dafd14c6217d7f83db6fae6f70c), blockNo= 0). UTxO state was added to the end.                                                                                                                                
    <similar lines of output omitted>
    
    Slot 00002: Wd3eddd0: InsertionSuccess: New tip is Tip(slot= Slot 2, blockId= BlockId(76be8b528d0075f7aae98d6fa57a6d3c83ae480a8469e668d7b0af968995ac71), blockNo= 1). UTxO state was added to the end.
    Final balances
    Wallet 1bc5f27d7b4e20083977418e839e429d00cc87f3: 
        {, ""}: 100000000
    Wallet 3a4778247ad35117d7c3150d194da389f3148f4a: 
        {, ""}: 100000000
    Wallet 4e76ce6b3f12c6cc5a6a2545f6770d2bcb360648: 
        {, ""}: 100000000
    Wallet 5f5a4f5f465580a5500b9a9cede7f4e014a37ea8: 
        {, ""}: 100000000
    Wallet 7ce812d7a4770bbf58004067665c3a48f28ddd58: 
        {, ""}: 100000000
    Wallet 872cb83b5ee40eb23bfdab1772660c822a48d491: 
        {, ""}: 100000000
    Wallet bdf8dbca0cadeb365480c6ec29ec746a2b85274f: 
        {, ""}: 100000000
    Wallet c19599f22890ced15c6a87222302109e83b78bdf: 
        {, ""}: 100000000
    Wallet c30efb78b4e272685c1f9f0c93787fd4b6743154: 
        {, ""}: 100000000
    Wallet d3eddd0d37989746b029a0e050386bc425363901: 
        {, ""}: 100000000

which gives information about the various blocks created at slots 00001 and 00002 and the final balances of the ten wallets.

If we need to specify an Emulator configuration we can use the related command `runEmulatorTraceIO'` (with a prime at the end).


### Simulation of Vesting contract

We run the Vesting contract discussed in Lesson 3 by first activating the endpoints of mentioned contract.  We assign handles to such endpoints:

    myTrace :: EmulatorTrace ()
    myTrace = do
        h1 <- activateContractWallet (knownWallet 1) endpoints
        h2 <- activateContractWallet (knownWallet 2) endpoints

We call the first handle as the "give" endpoint and give it corresponding parameters:

    callEndpoint @"give" h1 $ GiveParams
        { gpBeneficiary = mockWalletPaymentPubKeyHash $ knownWallet 2
        , gpDeadline    = slotToBeginPOSIXTime def 20
        , gpAmount      = 10000000
        }

After waiting for slot 20, we call the second handle as the "grab" endpoint, and as such we do not need to pass any meaningful parameters so we pass unit:

    callEndpoint @"grab" h2 ()

We wait for two more slots and log the final slot reached:

    s <- waitNSlots 2
    Extras.logInfo $ "reached " ++ show s

All this is executed with:

    > :m Trace
    Prelude Trace> 
    Prelude Trace> :t test
    test :: IO ()
    Prelude Trace> test

which results in large amounts of data includying the log lines and final balances:

    <...>
    Slot 00002: *** CONTRACT LOG: "made a gift of 10000000 lovelace to 80a4f45b56b88d1139da23bc4c3c75ec6d32943c087f250b86193ca7 with deadline POSIXTime {getPOSIXTime = 1596059111000}"                                                           <...>
    Slot 00022: *** USER LOG: reached Slot {getSlot = 22}
    Slot 00022: *** CONTRACT LOG: "collected gifts"
    <...>
    Final balances
    Wallet 1bc5f27d7b4e20083977418e839e429d00cc87f3: 
        {, ""}: 100000000
    Wallet 3a4778247ad35117d7c3150d194da389f3148f4a: 
        {, ""}: 100000000
    Wallet 4e76ce6b3f12c6cc5a6a2545f6770d2bcb360648: 
        {, ""}: 100000000
    Wallet 5f5a4f5f465580a5500b9a9cede7f4e014a37ea8: 
        {, ""}: 100000000
    Wallet 7ce812d7a4770bbf58004067665c3a48f28ddd58: 
        {, ""}: 109995870
    Wallet 872cb83b5ee40eb23bfdab1772660c822a48d491: 
        {, ""}: 89999990
    Wallet bdf8dbca0cadeb365480c6ec29ec746a2b85274f: 
        {, ""}: 100000000
    Wallet c19599f22890ced15c6a87222302109e83b78bdf: 
        {, ""}: 100000000
    Wallet c30efb78b4e272685c1f9f0c93787fd4b6743154: 
        {, ""}: 100000000
    Wallet d3eddd0d37989746b029a0e050386bc425363901: 
        {, ""}: 100000000

