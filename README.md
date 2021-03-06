# README - Verifying Non-Interference Properties For Software Patches

## UofM -Task III AMP Challenge Problem 6


This README will help guide you through running this tool and additonally provides an explanation about using this tool for Challenge Problem 6

See [Challenge Problem 6](#Challenge-Problem-6) for specifc details and a video walkthrough
#Overview
This repo contains the ongoing work to produce non-interference proofs for software patches, specifically targetting challenge problem 6.

**This repo is a subset of the broader effort to produce non-interference proofs from [https://github.com/eligoldweber/llvm_non_interference_proofs](https://github.com/eligoldweber/llvm_non_interference_proofs) (specifically this [branch](https://github.com/eligoldweber/llvm_non_interference_proofs/tree/challengeProblem6))**

Included in this repo are operational semantics for LLVM defined in [Dafny](https://github.com/dafny-lang/dafny). Additionally, there are defined instruction level state transitions for a subset of LLVM instructions. Lastly there is a collection of examples of how to represent an LLVM program in terms of the operational semantics in a state machine format, and proofs of safety for these programs.

This README outlines the steps necessary for building and verifying dafny files using [scons](https://scons.org/) or .NET on either a **supplied Docker image** or locally. 


# Running With Docker

Pull a created docker image or use the included Dockerfile to create an image with the appropriate dependencies and run in interactive mode to execute proofs:

* Start by cloning this repo
* The remaining commands can be executed in the cloned repo's root directory

### Build Image Locally
1. `docker build -t dafny_iron_patch .`
2. `docker run -it --rm -v [FULL/PATH/TO/CLONED/REPO]:/src dafny_iron_patch`


### Pull Image
1. `docker pull eligoldweber/llvm_non_interference:dafny_iron_patch_challenge6`
2. `docker run -it --rm -v [FULL/PATH/TO/CLONED/REPO]:/src eligoldweber/llvm_non_interference:dafny_iron_patch_challenge6`

    ***

3. `cd src`
4. Verify using scons: `scons --dafny-path=/opt/dafny --verify-root=./Dafny/examples/Challenge6/Challenge6Properties.i.dfy` 


# Running Locally

To verify these proofs locally, without Docker, you will need the following dependencies:

 1. .NET 5.0 SDK (available at `https://dotnet.microsoft.com/download`)
 2. Dafny v3.0.0 or higher (verifier, available at `https://github.com/dafny-lang/dafny`)
 3. python 2 or 3 (needed for running scons)
 4. scons (installable by running `pip install scons`)


# Verifying Dafny Files



This applies to verifying files locally or using docker. 

 * For Challenge Problem 6: `scons --dafny-path=/opt/dafny --verify-root=./Dafny/examples/Challenge6/Challenge6Properties.i.dfy` <br \> <br \> 

 * In general, use `scons --dafny-path=/path/to/directory/with/dafny/`
 
 
 > **Note:** Dafny is installed in `/opt/dafny` in the Docker image
 
 
 The default verification will verify most `.dfy` files in the `src/Dafny/examples` directory and all of their dependencies. (Including the proofs for Challenge Problem 6, but excluding `Challenge6PropertiesSideEffect.i.dfy`)

 > **Note:** The first time running this, it will take some time as it will verify all `.dfy` files and all of their dependencies. The verification result is cached in a corresponding `.vdfy` file and upon running `scons` again if a specific dependency has not been modified, the cached verification results will be reused at runtime. 
 
 To remove all cached verification results use: `./scripts/cleanCachedResults.sh`. <br /> <br />
 
 
 * To specify a `.dfy` file that you wish to verify use the following command line argument with scons `--verify-root=/path/to/dfy/file` [Note: this will also verify all dependencies of the specified file]
			
* Change the default timeout used during verification with: `--time-limit=[time in seconds]` 

* Turn off trace with: `--no-trace=1` 


To verify files without the help of the scons tool, you can use the following command: `dotnet /path/to/directory/with/dafny/ [dafny parameters] ie. /compile:0 /timeLimit:60 /trace /arith:5 noCheating:1] /path/to/dafnyFile`

## Understanding the output
If the proof goes through (which is the expectation, unless otherwise explicitly stated)

* The result of individual files should resemble something like: `Dafny program verifier finished with 8 verified, 0 errors`

* The result of running SCons: `scons: done building targets.`

If a file is unable to verified (for any assortment of reasons), the final output will look something like: `Dafny program verifier finished with 3 verified, 1 error` and there will be some sort of reason/hint as to where the proof failied, (ie. `Error: A postcondition might not hold on this return path.`)

# Challenge Problem 6

Link to video showing setup and explanation for challenge problem 6: [https://youtu.be/aCRA0xDVP3c](https://youtu.be/aCRA0xDVP3c)

The main files of interest are located in the following directory: `Dafny/examples/Challenge6`

This directory is broken down as follows:

* **Challenge6Code.s.dfy:** This is a trusted file that contains the Dafny representation of the LLVM code from Challenge problem 6 and defines what a valid starting state is. This file is created manually by transcribing the LLVM generated from the source code into the corresponding Dafny representation. Specifically it contains the LLVM code for the `write_encrypted(void)` function in logging.c -- this is the place where the encrypt call to OpenSSL is made and the crc and sha256 hashing exists. (In the future this manual task will be automated) <br /> <br />

* **Challenge6CodeLemmas(Vuln/Patch/PatchSideEffect).i.dfy:** These three files contain important helper lemmas for the final proof. The corresponding lemmas `unwrapPatchBehaviors()`, `unwrapVulnBehaviors()`, and `unwrapPatchSideEffectBehaviors()` serve as witness lemmas that "unwrap" the corresponding blocks of LLVM code and in an abstract manner walk through the state transisitons defined by the state machine. These are important because they demonstrate the set of possible behaviors and their outputs for these three cases. We then use this in proving properties, by reasioning about the set of possible behaviors. 

<!-- This is the main file associated with this challenge and contains the main proof. With the absence of a formal spec for the patch -- changing DES encryption out for AES, it becomes difficult to prove non-interference between the patched and unpatched versions of the code. This would entail showing that all "non-vulnerable" executions of the unpatched version are indistinguishable from the patched version. It is not possible to determine what "non-vulnerable" is in this context if the concern is a brute force attack. Rather, we aim to prove that when calling out to OpenSSL's encryption function there are no unexpected additional side effects. We prove that no other state is changed other than bytes_written and the CipherText after executing the encrypt function.  <br /> <br /> -->

* **ChallengeCommon.i.dfy:** This file contains additional lemmas that are used in Challenge6 in different modules and can be re-used from a common location  <br /> <br />

* **Challenge6Properties.i.dfy:** This file contains the main proof for this challenge. Specifcially we prove the benign patch property for this example - see [General Non Interference Properties](#General-Non-Interference-Properties). This file contains the lemma `patchIsBenign()` which ensures as a post condition that this property holds. By proving that the patch is benign we show that the patch does not add any NEW behaviors. Rather any 'valid' behavior in the patched system is nothing new, and could exist in the vulnerable version. Similar to Challenge Problem 5, it is not possible to specify the vulnerable executions between using crc vs sha256, as such we are limited to proving only the benign property for this patch. To define the MiniSpec used in the other properties we would need to define the set of behaviors that are vulnerable.  <br /> <br />

* **Challenge6PropertiesSideEffect.i.dfy:** This file serves as an example of the benefits of using this approach. In this case, verfying this file will result in a failure (timeout or assertion failure), this is because in this case we patch the code, but introduce an additional side effect modifying the `INTEGRITY_SIZE` constant during a hashing call, as such we introduce a new, and also incorrect, behavior into the set of patched behaviors. This violates the benign patch property and thus the proof catches this and fails.  <br /> <br />
 
Scope of the proof:

Due to complexites in modeling the behavior of the encrypt function and time constraints, the current state of Challenge Problem 6 makes the strong assumtion and models the encrpytion functions and hashing functions as the identity function. This essentially turns a call to the encrpytion function to a stutter step, where we assume that the state from before and after the call to encrypt() remains the same. We assume that the encryption function is correct and doesnt do anything else than what it is supposed to.  

## General Non Interference Properties

See `Dafny/Common/GeneralNonInterferenceProperties.s.dfy` for more details

To prove full non-interference we aim to prove 3 properties, and show that the conjection of them all result in non-interference. 

* **`benignPatch`:** The patch does not add any new behavior


```
    predicate benignPatch(pre:set<behavior>,post:set<behavior>)
    {
        forall postB :: postB in post ==> postB in pre
    }
```

* **`successfulPatch`:** The patch prunes the BAD (defined by MiniSpec) behaviors


```
    predicate successfulPatch(post:set<behavior>)
    {
        forall postB :: MiniSpec(postB) ==> !(postB in post)
    }
```


* **`completePatch`:** The patch preserves the GOOD behavior


```
    predicate completePatch(pre:set<behavior>,post:set<behavior>)
    {
        forall p :: (p in pre && !MiniSpec(p)) ==> p in post
    }
```

## Notes/Troubleshooting

If the following error occurs `ValueError : unsupported pickle protocol: 4` try deleting the `.sconsign.dblite` file (found in the repo's root directory) and run the command again. 


## Questions or Issues

If there are any questions or issues, please contact:

Eli Goldweber -- [edgoldwe@umich.edu](mailto:edgoldwe@umich.edu)




