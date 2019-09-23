---?image=assets/images/gitpitch-audience.jpg
@title[EDK_II_OpenBoard_Platform_pres]
<br><br><br>
<span style="font-size:0.75em" >This slide deck has moved to: https://gitpitch.com/tianocore-training/EDK_II_OpenBoard_Platform/master#/
</span>
<br><br>
## <span class="gold"   >UEFI & EDK II Training</span>
<span style="font-size:0.85em" >EDK II Open Board Platform Design for Intel Architecture(IA)


<br>
<span style="font-size:0.75em" ><a href='http://www.tianocore.org'>tianocore.org</a></span>
Note:
  PITCHME.md for UEFI / EDK II Training  EDK II OpenBoard Platform and Porting  

  Copyright (c) 2019, Intel Corporation. All rights reserved.<BR>

  Redistribution and use in source (original document form) and 'compiled'
  forms (converted to PDF, epub, HTML and other formats) with or without
  modification, are permitted provided that the following conditions are met:

  1) Redistributions of source code (original document form) must retain the
     above copyright notice, this list of conditions and the following
     disclaimer as the first lines of this file unmodified.

  2) Redistributions in compiled form (transformed to other DTDs, converted to
     PDF, epub, HTML and other formats) must reproduce the above copyright
     notice, this list of conditions and the following disclaimer in the
     documentation and/or other materials provided with the distribution.

  THIS DOCUMENTATION IS PROVIDED BY TIANOCORE PROJECT "AS IS" AND ANY EXPRESS OR
  IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF
  MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO
  EVENT SHALL TIANOCORE PROJECT  BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
  SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
  PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS;
  OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
  WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR
  OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS DOCUMENTATION, EVEN IF
  ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.



---  
@title[Lesson Objective]
<BR>
### <p align="center"<span class="gold"   >Lesson Objective </span></p>

<!---  Add bullets using https://fontawesome.com/cheatsheet certificate
-->
<ul style="list-style-type:none">
 <li>@fa[certificate gp-bullet-yellow]<span style="font-size:0.9em">&nbsp;&nbsp;Introduce Minimum Platform Architecture (MPA)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span> </li>
 <li>@fa[certificate gp-bullet-green]<span style="font-size:0.9em">&nbsp;&nbsp;Explain the EDK II Open board platforms <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;infrastructure  & focus areas</span> </li>
 <li>@fa[certificate gp-bullet-cyan]<span style="font-size:0.9em">&nbsp;&nbsp;Describe Intel® FSP with  the EDK II open board<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;platforms </span></li>
</ul>

---
@title[Current Issues ]
<p align="right"><span class="gold" >@size[1.1em](<b>Current Issues</b>)</span><br>
<span style="font-size:0.75em;" > Open Source EDK II Platforms</span></p>


@snap[north span-80 ]
<br>
<br>
<br>
@box[bg-royal text-white rounded my-box-pad2 fragment ](<p style="line-height:60%" align="left"><span style="font-size:0.75em;" >&nbsp;&nbsp;Developers need a way to turn on and off of a feature. <br>&nbsp;</span></p>)
@box[bg-royal text-white rounded my-box-pad2 fragment ](<p style="line-height:60%" align="left"><span style="font-size:0.75em;" >&nbsp;&nbsp;Developers need a way to get platform  <br>&nbsp;&nbsp;configuration data . <br>&nbsp;</span></p>)
@box[bg-royal text-white rounded my-box-pad2 fragment ](<p style="line-height:60%" align="left"><span style="font-size:0.75em;" >&nbsp;&nbsp;Developers need to do porting work from an <br>&nbsp;&nbsp;existing board to a new board  . <br>&nbsp;</span></p>)
@box[bg-royal text-white rounded my-box-pad2 fragment ](<p style="line-height:60%" align="left"><span style="font-size:0.75em;" >&nbsp;&nbsp;Developers might need to work on a different board . <br>&nbsp;</span></p>)
@snapend


Note:

Before an open source IA firmware appeared, there were many closed source IA firmware solutions in the world. We did research on the UEFI firmware examples of Intel ATOM based small core, Intel Core-i7 based big core client, and Intel XEON based big core servers. We came across some common issues, including: 

1) Developers need a way to turn on and off of a feature. 
For example, the Trusted Platform Module (TPM) or UEFI Secure Boot may need to be supported. In principle it is valuable to provide this class of capability, but the problem is too many configurations are often provided. In fact, we observed that one BIOS provides more than 100 configurations exposed for the developers to control. Regrettably, some configurations of those various controls do not even work. 
This often leads to the request of having a minimal BIOS to boot, and to do so, how to select among these 100 configurations? 


2) Developers need a way to get the platform configuration data. 
For example, one configuration choice can include “is VT enabled by the end user?” Another control can include “is the TSEG SMRAM size 1M, 8M or 16M,” or “is there an Embedded Controller (EC) or DOCK attached on the board?” The EDKII BIOS provides many choices on the source of the configuration data. For example, the UEFI specification defines UEFI Variables; the UEFI PI specification defines PCD; the Intel FSP defines UPD; silicon reference code defines the policy HOB, policy PPI, and policy protocol; silicon specific code has a signed static configuration data blob; and even legacy CMOS region exists. People may ask: which interface should I use in my platform code? 

3)
Developers need to do porting work from an existing board to a new board. 
There might be GPIO routing differences, or alternate component choices, such as SIO differences. However, some older platform code may use a “switch-case” mechanisms to check the board type, with such “switch-case” usages is scattered across many platform drivers. These platform elements include AcpiPlatform, SmmPlatform, PlatformInit, EC, ASL code, VFR pages, etc. In order to add a new board on update a existing platform, a developer has to find out all the places to make the change. 
People may think: How can I know how many modules I need to port, and when have I finished updating all required modules? 


4) Developers might need to work on a different board. 
For example, there might be an ATOM based on a server, a Core-i7 based server, or a XEON based server. However, the BIOS from different segments are different. We once compared an ATOM based firwmare with a Core-i7 based firmware. There are ~20 directories under Platform. Only 2 are same, which are “Include”, and “Library”. People might require significant time to ramp up again to get familiar with the new platform structure. 
Why can’t the platform tree structures bear more similarity? 

---?image=assets/images/binary-strings-black2.jpg
@title[Introducing MPA]
<br><br><br><br><br>
## <span class="gold"  >&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Introducing</span>
<span style="font-size:0.9em" > &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Minimum Platform Architecture (MPA) </span>


---
@title[Goals]
<p align="right"><span class="gold" >@size[1.1em](<b>GOALS</b>)</span><br>
<span style="font-size:0.85em;" ><b>Minimum Platform Architecture (MPA)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</b></span></p>


@snap[north-west span-30 ]
<br>
<br>
<br>
@box[bg-green-pp text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><b>Simple</b><br><br>&nbsp;</span></p>)
@box[bg-green-pp text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.9em;" ><b>Portable</b><br><br>&nbsp;</span></p>)
@box[bg-green-pp text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.9em;" ><b>Consistent</b><br><br>&nbsp;</span></p>)
@box[bg-green-pp text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.9em;" ><b>Code<br>Convergence</b><br>&nbsp;</span></p>)
@snapend


@snap[north-east span-67 ]
<br>
<br>
@css[text-white fragment](<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" ><br>Code structure should be obvious so that the firmware developer can easily turn on or turn off a significant feature<br></span></p>)
@css[text-white fragment](<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" >Firmware developer can easily port and enable a new board.<br><br> </span></p>)
@css[text-white fragment](<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" >Firmware code structure should be independent of processor/silicon architecture or platform type &lpar;embedded, workstation, server, etc.&rpar;<br></span></p>)
@css[text-white fragment](<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" >One instance of code per task</span></p>)
@snapend

@snap[south span-85 fragment]
@box[bg-purple-pp text-white rounded my-box-pad2  ](<p style="line-height:40%"><span style="font-size:0.8em">Design open source EDK II  Intel Architecture firmware<br><br>&nbsp;</span></p>)
@snapend

Note:

### Goals 
There is an existing myth that IA firmware is complex and hard to port or enable for a new platform. 
Goal is to provide some guidance on how to design open source EDK II  IA firmware solution


### Minimum Platform Objectives : 
- Simple - Define a structure that enables developers to consistently navigate source code, execution flow, and the functional results of bootstrapping a system.
- Portable - Enable a minimal platform where minimal is defined as the minimal firmware implementation required to produce a basic solution that can be further extended to meet a multitude of client, server, and embedded market needs.
- Consistent  - Enable large granularity binary solutions.
- Code Convergence - Minimize coupling between common, silicon, platform, and board packages.


### Code convergence - The most important aspect of Minimum Platform for Intel product delivery is code convergence. This simply means having one instance of code per task.

---?image=assets/images/slides/Slide6.JPG
@title[Code Convergence, WHY?]
<p align="right"><span class="gold" >@size[1.1em](<b>WHY?</b>)</span><br>
<span style="font-size:0.99em;" ><b>Code Convergence&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
</b></span></p>

@snap[north-east span-78 ]
<br>
<br>
<br>
<br>
@box[bg-grey-85-trans text-black rounded my-box-pad2 fragment ](<p style="line-height:60% "><span style="font-size:0.7em;" >System firmware (BIOS ) is the largest payload in the IFWI binary image  &nbsp;</span></p>)
@box[bg-grey-85-trans text-black rounded my-box-pad2 fragment ](<p style="line-height:60% "><span style="font-size:0.7em;" >Platform implementation is ~2-3 million lines+ of “C” code <br>&nbsp;</span></p>)
@box[bg-grey-85-trans text-black rounded my-box-pad2 fragment ](<p style="line-height:60% "><span style="font-size:0.7em;" >Technology complexity increasing, strains firmware implementation solutions &nbsp;</span></p>)
@box[bg-grey-85-trans text-black rounded my-box-pad2 fragment ](<p style="line-height:60% "><span style="font-size:0.7em;" >Limited firmware engineering recourses  <br>&nbsp;</span></p>)
@snapend


@snap[south span-85 fragment]
@box[bg-purple-pp text-white rounded my-box-pad2  ](<p style="line-height:40%"><span style="font-size:0.8em">Copy + Paste + Modify  = Human Errors<br><br>&nbsp;</span></p>)
@snapend

Note:
### Why?  Code Convergence is most important
- BIOS is the largest firmware payload in the IFWI (by binary size and source size)
- A typical Intel implementation is ~2-3 million lines+ of C code
- Technology complexity increases over time putting strain on firmware to keep up
- Engineering resources are always a constrained resource
 
The copy + paste + modify model used today is prone to human error and ripples update tasks across each platform team - this is more work than necessary.

- How often have you seen old product code (or even code names) that no longer apply to the new product stick around for months?
- How often have you seen a fix in the same code elsewhere not in the current platform code because the current platform code was copied earlier than the fix or not updated after the fix?
- How often have you understood a flow or code implementation then could not understand elsewhere because it is arbitrarily implemented differently?
- Have you thought about improving the code base then realized it would need to be merged so many other places. Often this is "too risky" or "there's not enough time to understand your change to merge it" or "there's already too many changes in the other platform so the merge turns into rewriting the change".
- These are real problems that lead to bugs, poor software quality, loss of engineering time to reverse engineer duplicate flow implementations, etc. Loss of engineering time is significant. Duplicate code affects bug triage/debug time, new platform feature implementation time and quality, training material creation/update work, presentation for change proposal work, etc.

---
@title[Move to Open Source]
<p align="right"><span class="gold" >@size[1.1em](<b>Why Move to Open Source ? </b>)</span><br>
<span style="font-size:0.75em;" >  </span></p>
<p style="line-height:70%" align="left" ><span style="font-size:0.85em;" >
@color[cyan](<b>Goal:</b>)
</span></p>
 <ul style="list-style-type:none; line-height:0.75;">
   <li>  <span style="font-size:0.75em;" >Make Intel products easy to work with and more secure. </span></li>
   <li>  <span style="font-size:0.75em;" >Do firmware development work directly in open source. </span></li>
 </ul>

<p style="line-height:70%" align="left" ><span style="font-size:0.85em;" >
@color[cyan](<b>Benefits:</b>)
</span></p>
 <ul style="list-style-type:none; line-height:0.75;">
   <li>  <span style="font-size:0.75em;" >Allow improved customer engagements </span></li>
   <li>  <span style="font-size:0.75em;" >Builds transparency and trust </span></li>
   <li>  <span style="font-size:0.75em;" >Reduce overhead to transition from internal to external </span></li>
   <li>  <span style="font-size:0.75em;" >Deploy fixes across the ecosystem more rapidly </span></li>
 </ul>


@snap[south span-85 fragment]
@box[bg-purple-pp text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.8em">Easier to access, understand, fix & optimize means improved product quality <br>&nbsp;</span></p>)
@snapend


Note:
### Open source 
- As firmware engineers we need to make Intel products easy to work with and keep more secure. 
### Benefits:
- Open source allows us to improve customer engagement with all customers, build transparency and trust, reduce overhead translating internal and external process, and deploy fixes across the ecosystem more rapidly.

- Main point: Open source is not more work if you do your work in open source. This may be hard to imagine because today's code is too monolithic for this to be possible. A software architecture that allows modular pieces to be integrated from open source into closed source solutions is necessary.

- The easier we make it to access, understand, fix, improve, and optimize Intel firmware (ideally through single instance, high quality code) we can better engage with our customers building real world solutions to improve Intel product quality (all Intel products including those without EDK II firmware).



---?image=assets/images/slides/Slide8.JPG
@title[Four Focus Areas Section]
<br>
<p align="left"><span class="gold" >@size[1.1em](<b>Four Focus Areas</b>)</span></span></p>

@snap[north-east span-35 fragment]
<br>
<br>

<ul style="list-style-type:disc; line-height:0.7;">
  <li><span style="font-size:0.65em" >Minimal /Full BIOS </span> </li>
  <li><span style="font-size:0.65em" >Feature ON/OFF </span> </li>
  <ul style="list-style-type:disc; line-height:0.6;">
  <li><span style="font-size:0.6em" >SMBIOS</span> </li>
  <li><span style="font-size:0.6em" >TPM </span> </li>
  <li><span style="font-size:0.6em" >Secure Boot </span> </li>
  <li><span style="font-size:0.6em" >. . . </span> </li>
  </ul>
</ul>
@snapend

@snap[south-west span-30 fragment]
<br>
<ul style="list-style-type:disc; line-height:0.7;">
  <li><span style="font-size:0.65em" >Setup Variable</span> </li>
  <li><span style="font-size:0.65em" >PCD </span> </li>
  <li><span style="font-size:0.65em" >Policy Hob/PPI/Protocol </span> </li>
</ul>
<br>
<br>
<br>
@snapend

@snap[south-east span-33 fragment]
<br>
<ul style="list-style-type:disc; line-height:0.7;">
  <li><span style="font-size:0.65em" >GPIO </span> </li>
  <li><span style="font-size:0.65em" >SIO </span> </li>
  <li><span style="font-size:0.65em" >ACPI </span> </li>
  <li><span style="font-size:0.65em" >Stages </span> </li>
</ul>
<br>
<br>
<br>
<br>
@snapend



Note:
In order to provide suggestions on the problem statements earilier, we need to focus on the following four areas: 

- Feature. How does a BIOS provide the feature selection option to a developer? 
- Configuration. From which interface can a platform module get the configuration data? 
- Porting. Where are the modules to be ported for a new board? 
- Tree Structure. What does an EDKII platform package look like? 


---?image=assets/images/slides/Slide9.JPG
@title[Tree Structure Section]
<br>
<p align="left"><span class="gold" >@size[1.1em](<b>Tree Structure</b>)</span></span></p>


Note:
Tree Structure. What does an EDK II platform package look like? 


This is the directory structure of our EDK II platform in relationship to the whole and other areas, such as the boot flow, or kernel, or core.

---?image=assets/images/slides/Slide10.JPG
@title[Organization]
<p align="right"><span class="gold" >@size[1.1em](<b>Organization</b>)</span></span></p>


@snap[north span-54 ]
<p style="line-height:10%" align="left" ><span style="font-size:0.7em;" ><br><br>&nbsp;
</span></p>

@box[bg-grey-15 text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.9em;" ><b>&nbsp;</b><br><br>&nbsp;</span></p>)
@box[bg-grey-15 text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.9em;" ><b>&nbsp;</b><br><br>&nbsp;</span></p>)
@box[bg-grey-15 text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.9em;" ><b>&nbsp;</b><br><br>&nbsp;</span></p>)
@box[bg-grey-15 text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.9em;" ><b>&nbsp;</b><br><br>&nbsp;</span></p>)
@box[bg-grey-15 text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.9em;" ><b>&nbsp;</b><br><br>&nbsp;</span></p>)
@snapend


@snap[north-west span-30 ]
<p style="line-height:10%" align="left" ><span style="font-size:0.7em;" ><br><br>&nbsp;
</span></p>

@box[bg-gold2 text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><b>Common</b><br><br>&nbsp;</span></p>)
@box[bg-gold2 text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.9em;" ><b>Platform</b><br><br>&nbsp;</span></p>)
@box[bg-gold2 text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.9em;" ><b>Board</b><br><br>&nbsp;</span></p>)
@box[bg-gold2 text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.9em;" ><b>Silicon</b><br><br>&nbsp;</span></p>)
@box[bg-gold2 text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.9em;" ><b>Features</b><br><br>&nbsp;</span></p>)

@snapend



@snap[north-east span-67 ]
<p style="line-height:10%" align="left" ><span style="font-size:0.7em;" ><br><br>&nbsp;
</span></p>

@css[text-white fragment](<p style="line-height:60%" align="left" ><span style="font-size:0.67em;" >&bull;<br> No direct HW requirements<br><br><br></span></p>)
@css[text-white fragment](<p style="line-height:60%" align="left" ><span style="font-size:0.67em;" >&bull; Enable a specific <br>&nbsp;&nbsp;&nbsp;platform's capabilities <br><br><br> </span></p>)
@css[text-white fragment](<p style="line-height:60%" align="left" ><span style="font-size:0.67em;" >&bull; Board specific code <br><br><br><br> </span></p>)
@css[text-white fragment](<p style="line-height:60%" align="left" ><span style="font-size:0.67em;" >&bull; Hardware specific code </span></p>)
@css[text-white fragment](<p style="line-height:60%" align="left" ><span style="font-size:0.67em;" ><br>&bull;Advanced features, non-essential<br>&nbsp;&nbsp;&nbsp; for "basic OS boot" </span></p>)
@snapend


Note:
The architecture makes use of four primary classifications of code that are generally instantiated in different EDK II packages.
- Common (EDK II) is code that does not have any direct HW requirements other than the basics required to execute machine code on the processor (stack, memory, IA registers, etc).
   - Producer(s): TianoCore.org


- Platform defines the actions needed to enable a specific platform's capabilities. In this architecture, capabilities are divided into mandatory and advanced features. Mandatory features are enabled in stages prior to Stage VI. Advanced features are enabled in Stage VI and later.
  - Minimum Platform Producer(s): TianoCore.org
  - Advance Feature Producer(s): TianoCore.org, OEM, BIOS vendor
  - Board packages contains board specific code for one or more motherboards.
  - Producer(s): Device manufacturer, BIOS vendor, Board user

- Silicon, also often called hardware code, has some tie to a specific class of physical hardware. Sometimes governed by industry standards, sometimes proprietary. Silicon or hardware code is usually not intended to have multiple implementations for the same hardware.
  - Producer(s): Silicon vendor

---
blank slide

---?image=assets/images/slides/Slide7.JPG
@title[Open Source EDK II Workspace]
<p align="right"><span class="gold" >@size[1.1em](<b>Open Source EDK II Workspace</b>)</span></span></p>

@snap[north-west span-50 ]
<br>
<br>
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend


@snap[north span-30 ]
<br>
<br>
@box[bg-gold2 text-white rounded my-box-pad2 fragment ](<p style="line-height:60% "><span style="font-size:0.9em;" ><b>Common</b><br><br>&nbsp;</span></p>)
@box[bg-gold2 text-white rounded my-box-pad2 fragment ](<p style="line-height:60%"><span style="font-size:0.9em;" ><b>Platform</b><br><br>&nbsp;</span></p>)
@box[bg-gold2 text-white rounded my-box-pad2 fragment ](<p style="line-height:60%"><span style="font-size:0.9em;" ><b>Board</b><br><br>&nbsp;</span></p>)
@box[bg-gold2 text-white rounded my-box-pad2 fragment ](<p style="line-height:60%"><span style="font-size:0.9em;" ><b>Silicon</b><br><br>&nbsp;</span></p>)
@snapend



@snap[north-west span-60 ]
<br>
<br>
<p style="line-height:50%" align="left" ><span style="font-size:0.5em; font-family:Consolas;">
&nbsp;MyWorkSpace/<br>&nbsp;&nbsp;
@color[yellow](edk2)/<br>&nbsp;&nbsp;&nbsp;&nbsp;
  - "@color[#FFC000](<font face="Arial">edk2 Common</font>)"<br>&nbsp;&nbsp;
@color[yellow](edk2-platforms)/<br>&nbsp;&nbsp;&nbsp;&nbsp;
  Platform/ "@color[#FFC000](<font face="Arial">Platform</font>)"<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     Intel/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
       MinPlatformPkg/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
          BoardX/ “@color[#FFC000](<font face="Arial">Board</font>)”<br>&nbsp;&nbsp;&nbsp;&nbsp;
  Silicon/ "@color[#FFC000](<font face="Arial">Silicon</font>)"<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     Intel/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
       MinPlatformPkg/<br>&nbsp;&nbsp;
@color[yellow](edk2-non-osi)/<br>&nbsp;&nbsp;&nbsp;&nbsp;
  Silicon/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     Intel/<br>&nbsp;&nbsp;
@color[yellow](FSP)/"@color[#FFC000](<font face="Arial">Silicon</font>)"<br>&nbsp;&nbsp;&nbsp;&nbsp;
   . . ./<br>&nbsp;&nbsp;
</span></p>
@snapend


Note:
The build process creates
   this directory -> Build/
MyWorkSpace – directory from the “git” of repositories

Build –p .dsc from the BOARD Directory

The architecture is designed to support a maintainer ownership model. For example, board developers should not directly modify (fork) the platform, silicon, or common code. More details on conventional usage of the package classifications can be found in supplemental literature from UEFI Forum, TianoCore.org, and others.


---?image=assets/images/slides/Slide7.JPG
@title[Open Board Tree Structure]
<p align="right"><span class="gold" >@size[1.1em](<b>Open Board Tree Structure</b>)</span></span></p>

@snap[north-west span-75 ]
<br>
<br>
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend



@snap[north-west span-70 ]
<br>
<br>
<p style="line-height:50%" align="left" ><span style="font-size:0.5em; font-family:Consolas;">
&nbsp;&nbsp;
@color[yellow](edk2)/  <a href="https://github.com/tianocore/edk2"> github.com/edk2</a><br>&nbsp;&nbsp;&nbsp;&nbsp;
. . .<br>&nbsp;&nbsp;
@color[yellow](edk2-platforms)/  <a href="https://github.com/tianocore/edk2-platforms"> github.com/edk2-platforms</a><br>&nbsp;&nbsp;&nbsp;&nbsp;
  Platform/ <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     Intel/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
       AdvancedFeaturePkg/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
       KabylakeOpenBoardPkg/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
          KabylakeRvp3/ <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
       MinPlatformPkg/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
       Vlv2TbltDevicePkg/<br>&nbsp;&nbsp;&nbsp;&nbsp;
  Silicon/ <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     Intel/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
       KabylakeSiliconPkg/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
       &nbsp;. &nbsp;. &nbsp;./<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
       Vlv2DeviceRefCodePkg/<br>&nbsp;&nbsp;
@color[yellow](edk2-non-osi)/<a href="https://github.com/tianocore/edk2-non-osi/tree/devel-MinPlatform">github.com/edk2-non-osi</a><br>&nbsp;&nbsp;&nbsp;&nbsp;
  Silicon/<br>&nbsp;&nbsp;
@color[yellow](FSP)/<a href="https://github.com/IntelFsp/FSP">github.com/Intel/FSP</a><br>&nbsp;&nbsp;&nbsp;&nbsp;
   KabylakeFspBinPkg/<br>&nbsp;&nbsp;
</span></p>
@snapend



@snap[north-east span-60 fragment ]
<br>
<br>
<p style="line-height:50%" align="left" ><span style="font-size:0.5em; font-family:Consolas;">
@color[#A8ff60](&larr;)&nbsp;@color[#FFC000](<font face="Arial">Common</font>)<br><br>
<br>
<br>
<br>
<br>
@color[#A8ff60](&larr;)&nbsp;@color[#FFC000](<font face="Arial">Platform</font>)<br>
@color[#A8ff60](&larr;)&nbsp;@color[#FFC000](<font face="Arial">Board</font>) <br>
@color[#A8ff60](&larr;)&nbsp;@color[#FFC000](<font face="Arial">Common</font>)<br>
<br>
<br>
<br>
@color[#A8ff60](&larr;)&nbsp;@color[#FFC000](<font face="Arial">Silicon</font>)<br>
<br>
<br>
<br>
<br>
<br>
@color[#A8ff60](&larr;)&nbsp;@color[#FFC000](<font face="Arial">Silicon</font>)<br>&nbsp;&nbsp;
</span></p>
@snapend


Note:
This is an example of the MinPlatform for the Intel Architecture (IA)
With Kabylake and  Vlv2TbltDevicePkg (Minnowboard MAX) as examples


Notice the 3 different repositories for the different sources align similarly to the COMMON, PLATFORM, BOARD & SILICON layout

Not shown is the edk2 repository since this should always be considered as common

---
@title[Directory Description]
<p align="right"><span class="gold" >@size[1.1em](<b>Directory Description</b>)</span></span></p>
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" ><b>@color[yellow](edk2-platform): </b>EDK II repo includes open source platform code </span></p>

<ul style="list-style-type:disc; line-height:0.7;">
  <li><span style="font-size:0.65em" >Platform folder : contains the platform specific modules by architecture</span> </li>
  <ul style="list-style-type:disc; line-height:0.7;">
    <li><span style="font-size:0.65em" >@color[cyan](MinPlatformPkg ): generic platform instance to control the boot flow.  </span> </li>
    <li><span style="font-size:0.65em" >@color[cyan](AdvancedFeaturePkg ): package to hold the advanced platform features </span> </li>
    <li><span style="font-size:0.65em" >@color[cyan](&lt;Generation&gt;OpenBoardPkg ): the silicon generation specific board package. All of the boards based upon this silicon generation can be located here </span> </li>
 </ul>
  <li><span style="font-size:0.65em" >Silicon folder : contains the silicon specific modules </span> </li>
  <ul style="list-style-type:disc; line-height:0.7;">
    <li><span style="font-size:0.65em" >@color[cyan](&lt;Generation&gt;SiliconPkg ): the silicon generation specific silicon package </span> </li>
  </ul>
</ul>

<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" ><b>@color[yellow](edk2-non-osi):</b> 
EDK II repo for platform modules in binary format(ex: silicon init binaries).
</span></p>
<ul style="list-style-type:disc; line-height:0.7;">
  <li><span style="font-size:0.65em" >@color[cyan](&lt;Generation&gt;SiliconBinPkg ):It is the silicon generation specific binary package. For example, CPU Microcode or the silicon binary FVs</span> </li>
</ul>


@snap[south span-95 fragment]
@box[bg-purple-pp text-white rounded my-box-pad2  ](<p style="line-height:40%"><span style="font-size:0.8em">&nbsp;Ideally, Only &lt;Generation&gt;OpenBoardPkg needs updating<br><br>&nbsp;</span></p>)
@snapend

Note:
- edk2-platform: EDK II repo to include any open source platform code, which has an  EDK II compatible license.
  - Platform folder: It contains the platform specific modules.
    - MinPlatformPkg: It is a generic platform instance to control the boot flow.
    - AdvancedFeaturePkg: It is a package to hold the advanced platform features. These features are not required for a basic OS boot, but they may be needed to provide a full feature platform firmware.
    - <Generation>OpenBoardPkg: It is the silicon generation specific board package. All of the boards based upon this silicon generation can be located here.
  - Silicon folder: It contains the silicon specific modules.
    - <Generation>SiliconPkg: It is the silicon generation specific silicon package.
- edk2-non-osi: It is the EDKII repo to include any open platform modules which is in a binary format, such as FSP binary, or CPU microcode.
  - <Generation>SiliconBinPkg: It is the silicon generation specific binary package. For example, CPU Microcode or the silicon binary FVs.

---
@title[FSP Directory Description]
<p align="right"><span class="gold" >@size[1.1em](<b>FSP Directory Description</b>)</span></span></p>
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" ><b>@color[yellow](FSP): </b>repo for Intel® Firmware Support Package (FSP) binaries </span></p>

<ul style="list-style-type:disc; line-height:0.8;">
  <li><span style="font-size:0.65em" >Platform folder Pkg : Each FSP project will be hosted in a separate directory</span> </li>
  <li><span style="font-size:0.65em" >ApolloLakeFspBinPkg Intel® Atom™ processor E3900 product family </span> </li>
  <li><span style="font-size:0.65em" >&nbsp;. &nbsp;. &nbsp;. </span> </li>
  <li><span style="font-size:0.65em" >CoffeeLakeFspBinPkg - 8th Generation Intel® Core™ processors and chipsets @size[.7em]( - formerly Coffee Lake and Whiskey Lake)<br> </span> </li>
  <li><span style="font-size:0.65em" >@color[cyan](KabylakeFspBinPkg ): 7th Generation Intel® Core™ processors and chipsets</span> </li>
  <ul style="list-style-type:disc; line-height:0.7;">
    <li><span style="font-size:0.65em" >Include </span> </li>
    <ul style="list-style-type:none; line-height:0.7;">
       <li><span style="font-size:0.65em" > - FSP UPD structure and related definitions used with EDK II build </span> </li>
	</ul>
    <li><span style="font-size:0.65em" >Doc - Integration Guide .PDF documentation </span> </li>
    <li><span style="font-size:0.65em" >@color[cyan](FSP.fd ) -Binary to be included with flash device image </span> </li>
    <li><span style="font-size:0.65em" >FSP.bsf - Configuration File with IDE configuration tool </span> </li>

  </ul>

</ul>


@snap[south span-85 fragment]
@box[bg-purple-pp text-white rounded my-box-pad2  ](<p style="line-height:40%"><span style="font-size:0.8em">FSP each project based on Intel Architecture <br><br>&nbsp;</span></p>)
@snapend

Note:

Intel® Firmware Support Package (Intel® FSP) includes: 
 A binary file 
 An integration guide 
 A rebasing tool 
 An IDE configuration tool / Boot Setting File (BSF) 


---
@title[Board Package Structure ]
<p align="right"><span class="gold" >@size[1.1em](<b>Board Package Structure </b>)</span><span style="font-size:0.8em;" ><br>- <font face="Consolas">MinPlatformPkg</font></span></p>

@snap[north-west span-45 ]
<br>
<br>
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br>&nbsp;</span></p>)
@snapend

@snap[north-west span-100 ]
<br>
<p style="line-height:40% "><span style="font-size:0.5em; font-family:Consolas;" ><br>&nbsp;&nbsp;
@color[cyan](MinPlatformPkg)  /<br>&nbsp;&nbsp;&nbsp;&nbsp;
  <font face="Arial">&lt;Basic Common Feature&gt;/</font><br>&nbsp;&nbsp;&nbsp;&nbsp;
  Include /<br>&nbsp;&nbsp;&nbsp;&nbsp;
  Library /<br>&nbsp;&nbsp;&nbsp;&nbsp;
  PlatformInit /<br>&nbsp;&nbsp;&nbsp;&nbsp;
</span></p>

<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" >Where: </span></p>

<ul style="list-style-type:disc; line-height:0.7;">
  <li><span style="font-size:0.65em" >@color[yellow](&lt;Basic Common Feature&gt; ): The basic features to support OS boot, such as ACPI, flash, and FspWrapper. It also includes the basic security features such as Hardware Security Test  Interface (HSTI). </span> </li>
  <li><span style="font-size:0.65em" >@color[yellow](Include ): The include file as the package interface. All interfaces defined in MinPlatformPkg.dec are put to here.  </span> </li>
  <li><span style="font-size:0.65em" >@color[yellow](Library ): It only contains feature independent library, such as PeiLib. If a library is related to a feature, this library is put to <Feature>/Library folder, instead of root Library folder. </span> </li>
  <li><span style="font-size:0.65em" >@color[yellow](PlatformInit ): The common platform initialization module. There is PreMemPEI, PostMemPEI, DXE and SMM version. These modules control boot flow and provide some hook point to let board code do initialization. </span> </li>
</ul>
@snapend

@snap[north-east span-40 fragment]
<br>
<br>
<br>

@box[bg-purple-pp text-white rounded my-box-pad2  ](<p style="line-height:80%"><span style="font-size:0.8em">Basic Common Features<br>&nbsp;</span></p>)
@snapend

Note:

---
@title[Advanced Feature Package ]
<p align="right"><span class="gold" >@size[1.1em](<b>Advanced Feature Package </b>)</span><span style="font-size:0.8em;" ><br></span></p>

@snap[north-west span-45 ]
<br>
<br>
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br>&nbsp;</span></p>)
@snapend

@snap[north-west span-100 ]
<br>
<p style="line-height:40% "><span style="font-size:0.5em; font-family:Consolas;" ><br>&nbsp;&nbsp;
@color[cyan](AdvancedFeaturePkg)  /<br>&nbsp;&nbsp;&nbsp;&nbsp;
  &lt;AdvancedFeatureCommonFeature&gt;/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  Include /<br>&nbsp;&nbsp;&nbsp;&nbsp;
  Include /
</span></p>
<br>
<br>

<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" >Where: </span></p>

<ul style="list-style-type:disc; line-height:0.7;">
  <li><span style="font-size:0.65em" >@color[yellow](&lt;AdvancedCommonFeature&gt; ): The advanced features, such as SMBIOS table, IPMI </span> </li>
  <li><span style="font-size:0.65em" >@color[yellow](Include ): The include file as the package interface.   </span> </li>
</ul>
@snapend



Note:


---
@title[Open Board Package Structure ]
<p align="right"><span class="gold" >@size[1.1em](<b>Open Board Package Structure </b>)</span><span style="font-size:0.8em;" ><br></span></p>

@snap[north-west span-45 ]
<br>
<br>
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend

@snap[north-west span-100 ]
<br>
<p style="line-height:40% "><span style="font-size:0.5em; font-family:Consolas;" ><br>&nbsp;&nbsp;
@color[cyan](&lt;Generation&gt;OpenBoardPkg)  /<br>&nbsp;&nbsp;&nbsp;&nbsp;
  &lt;BasicCommonBoardFeature&gt;/<br>&nbsp;&nbsp;&nbsp;&nbsp;
  Include /<br>&nbsp;&nbsp;&nbsp;&nbsp;
  Library /<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  &lt;AdvancedCommonBoardFeature&gt; /<br>&nbsp;&nbsp;&nbsp;&nbsp;
  @color[cyan](&lt;Board&gt;) /<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    Include /<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    Library /<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    &lt;BoardSpecificFeature&gt; /<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    @color[cyan](OpenBoardPkg.dsc)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    @color[cyan](OpenBoardPkg.fdf)<br><br>
</span></p>

<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" >Where: </span></p>

<ul style="list-style-type:disc; line-height:0.7;">
  <li><span style="font-size:0.65em" >@color[yellow](&lt;BasicCommonBoardFeature&gt; ) and @color[yellow](&lt;AdvancedCommonBoardFeature&gt; ): designate a board generation specific feature. They need to be updated when we enable a board generation. </span> </li>
  <li><span style="font-size:0.65em" >@color[yellow](&lt;Board&gt; ): contains all the board specific settings. If we need to port a new board in this generation, copy the &lt;Board&gt; folder and update the copy’s settings
  </span> </li>
</ul>
@snapend



Note:


---
@title[One Feature, one directory Guideline]
<p align="right"><span class="gold" >@size[1.1em](<b>One Feature, One directory Guideline </b>)</span><span style="font-size:0.8em;" ><br></span></p>

@snap[north-west span-100 ]
<br>
<br>
<br>
<br>
<br>
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend

@snap[north-west span-100 ]
<br>
<br>
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" >Use a hierarchical layout , <font face="Consolas">KabylakeOpenBoardPkg</font> example</span></p>
@snapend

@snap[north-west span-45 ]
<br>
<br>
<br>
<br>
<p style="line-height:40% "><span style="font-size:0.5em; font-family:Consolas;" ><br>&nbsp;&nbsp;
KabylakeOpenBoardPkg  /<br>&nbsp;&nbsp;&nbsp;&nbsp;
  Acpi /<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    BoardAcpiDxe /<br>&nbsp;&nbsp;&nbsp;&nbsp;
  FspWapper /<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    Library /<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    PeiFspPolicyUpdateLib /<br>&nbsp;&nbsp;&nbsp;&nbsp;
  @color[cyan](KabylakeRvp3) /<br>&nbsp;&nbsp;&nbsp;&nbsp;
  Library /<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    BaseGpioExpanderLib / <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    PeiI2cAccessLib  /<br>&nbsp;&nbsp;&nbsp;&nbsp;
  Policy /<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    PolicyInitDxe /
</span></p>
@snapend


@snap[north-east span-55 ]
<br>
<br>
<br>
<br>
<p style="line-height:40% " align="left"><span style="font-size:0.5em; font-family:Consolas;" ><br>&nbsp;&nbsp;
  @color[cyan](KabylakeRvp3) / <font face="Arial"> &nbsp;&nbsp;Cont.</font><br>&nbsp;&nbsp;&nbsp;&nbsp;
    Library /<br>&nbsp;&nbsp;&nbsp;&nbsp;
    Include /<br>&nbsp;&nbsp;&nbsp;&nbsp;
    OpenBoardPkg.dsc  <br>&nbsp;&nbsp;&nbsp;&nbsp;
    OpenBoardPkg.fdf
</span></p>
@snapend


@snap[north-east span-55 fragment]
<br>
<br>
<br>
<br>
<p style="line-height:40% " align="left"><span style="font-size:0.5em; font-family:Consolas;" ><br>&nbsp;&nbsp;
  <br>&nbsp;&nbsp;&nbsp;&nbsp;
  <br>&nbsp;&nbsp;&nbsp;&nbsp;
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; @fa[long-arrow-alt-left fa-3x gp-bullet-yellow]
	<br>&nbsp;&nbsp;&nbsp;&nbsp;
   
</span></p>
@snapend

@snap[south span-85 fragment]
@box[bg-purple-pp text-white rounded my-box-pad2  ](<p style="line-height:40%"><span style="font-size:0.8em">Only put the basic features into the root directory<br><br>&nbsp;</span></p>)
<br>
@snapend


Note:

Use a hierarchical layout - Specifically, only put the basic features into the root directory and put the advanced features into a “Features” directory. 

Slide shows the KabylakeOpenBoardPkg. 
The common board related ACPI is in Acpi directory. The common board related FSP policy update is in FspWrapper. The library folder includes the board specific GpioExpanderLib and I2cAccessLib. They might be used for other Kabylake generation board. 

The KabylakeRvp3 folder contains all RVP3 related settings, such as GPIO, High Definition Audio (HAD) verb Table, HsioPtss table, SPD table. This folder also has DSC and FDF file. We can build a KabylakeRvp3 binary inside of this folder 

---
@title[Compare to MinnowBoard  MAX/Turbot]
<p align="right"><span class="gold" >@size[1.1em](<b>Compare to MinnowBoard  MAX/Turbot</b>)</span><span style="font-size:0.8em;" ><br></span></p>

@snap[north-west span-100 ]
<br>
<br>
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend



@snap[north-west span-35 ]
<br>
<p style="line-height:40% "><span style="font-size:0.5em; font-family:Consolas;" ><br>&nbsp;&nbsp;
@color[cyan](Vlv2TbltDevicePkg)  /<br>&nbsp;&nbsp;&nbsp;&nbsp;
   AcpiPlatform/<br>&nbsp;&nbsp;&nbsp;&nbsp;
   Application /<br>&nbsp;&nbsp;&nbsp;&nbsp;
   BootScriptSaveDxe /<br>&nbsp;&nbsp;&nbsp;&nbsp;
   FspAzaliaConfigData /<br>&nbsp;&nbsp;&nbsp;&nbsp;
   FspSupport /<br>&nbsp;&nbsp;&nbsp;&nbsp;
   FvbRuntimeDxe /<br>&nbsp;&nbsp;&nbsp;&nbsp;
   FvInfoPei  /<br>&nbsp;&nbsp;&nbsp;&nbsp;
   Include /<br>&nbsp;&nbsp;&nbsp;&nbsp;
   IntelGopDepex /<br>&nbsp;&nbsp;&nbsp;&nbsp;
   Library /<br>&nbsp;&nbsp;&nbsp;&nbsp;
   Logo /<br>&nbsp;&nbsp;&nbsp;&nbsp;
   Metronome /<br>&nbsp;&nbsp;&nbsp;&nbsp;
   MonoStatusCode /
</span></p>
@snapend

@snap[north span-35 ]
<br>
<p style="line-height:40% " align="left"><span style="font-size:0.5em; font-family:Consolas;" ><br>&nbsp;&nbsp;
  Override /<br>&nbsp;&nbsp;
  PciPlatform /<br>&nbsp;&nbsp;
  PlatformCpuInfoDxe /<br>&nbsp;&nbsp;
  PlatformDxe /<br>&nbsp;&nbsp;
  PlatformGopPolicy /<br>&nbsp;&nbsp;
  PlatformInfoDxe /<br>&nbsp;&nbsp;
  PlatformInitPei /<br>&nbsp;&nbsp;
  PlatformPei /<br>&nbsp;&nbsp;
  PlatformSetupDxe /<br>&nbsp;&nbsp;
  PlatformSmm /<br>&nbsp;&nbsp;
  PpmPolicy /<br>&nbsp;&nbsp;
  SaveMemoryConfig /<br>&nbsp;&nbsp;
  SmBiosMiscDxe /<br>&nbsp;&nbsp;
</span></p>
@snapend


@snap[north-east span-35 ]
<br>
<p style="line-height:40% " align="left"><span style="font-size:0.5em; font-family:Consolas;" ><br>&nbsp;&nbsp;
  SmmSwDispatch2OnSmm /<br>&nbsp;&nbsp;
  SwDispatchThunk /<br>&nbsp;&nbsp;
  SmramSaveInfoHandlerSmm /<br>&nbsp;&nbsp;
  Stitch /<br>&nbsp;&nbsp;
  UiApp /<br>&nbsp;&nbsp;
  VlvPlatformInitDxe /<br>&nbsp;&nbsp;
  Wpce791 /<br>&nbsp;&nbsp;
<br>&nbsp;&nbsp;
  PlatformPkg.dec  <br>&nbsp;&nbsp;
  PlatformPkg.dsc  <br>&nbsp;&nbsp;
  PlatformPkg.fdf
</span></p>
@snapend


@snap[north-east span-25 fragment]
<br>
<p style="line-height:40% " align="left"><span style="font-size:0.5em; font-family:Consolas;" ><br>&nbsp;&nbsp;
  <br>
  <br>
  <br>
  <br>
  <br>
  <br>
  <br>
  <br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; @fa[long-arrow-alt-left fa-3x gp-bullet-yellow]
  
</span></p>
@snapend

@snap[south span-85 fragment]
@box[bg-purple-pp text-white rounded my-box-pad2  ](<p style="line-height:40%"><span style="font-size:0.8em">This introduces issues when searching for a driver<br><br>&nbsp;</span></p>)
<br>
@snapend


Note:
This introduces issues when the developer wants to find a  particular driver. 


---?image=assets/images/slides/Slide17.JPG
@title[Focus - Features Section]
<br>
<p align="left"><span class="gold" >@size[1.1em](<b>Features</b>)</span></span></p>

@snap[north-east span-35 ]
<br>
<br>

<ul style="list-style-type:disc; line-height:0.7;">
  <li><span style="font-size:0.65em" >Minimal /Full BIOS </span> </li>
  <li><span style="font-size:0.65em" >Feature ON/OFF </span> </li>
  <ul style="list-style-type:disc; line-height:0.6;">
  <li><span style="font-size:0.6em" >SMBIOS</span> </li>
  <li><span style="font-size:0.6em" >TPM </span> </li>
  <li><span style="font-size:0.6em" >Secure Boot </span> </li>
  <li><span style="font-size:0.6em" >. . . </span> </li>
  </ul>
</ul>
@snapend





Note:
### Features

In order to provide suggestions on the problem statements above, we would like to focus on the following four areas: 
- Feature. How does a BIOS provide the feature selection option to a developer? 
- Configuration. From which interface can a platform module get the configuration data? 
- Porting. Where are the modules to be ported for a new board? 
- Tree Structure. What does an EDKII platform package look like? 


---?image=assets/images/slides/Slide18.JPG
@title[Feature – BIOS Module Selection]
<p align="right"><span class="gold" >@size[1.1em](<b>Feature – BIOS Module Selection</b>)</span><span style="font-size:0.8em;" ><br></span></p>
<p style="line-height:40% " align="left"><span style="font-size:0.9em;" >Minimum set of features based on Categories </span></p>

@snap[north span-50 ]
<br>
<br>

<p style="line-height:20% " align="left"><span style="font-size:0.9em;" ><br></span></p>
@box[bg-grey-15 text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.8em;" ><b>&nbsp;</b><br>&nbsp;</span></p>)
<p style="line-height:20% " align="left"><span style="font-size:0.9em;" ><br><br></span></p>
@box[bg-grey-15 text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.8em;" ><b>&nbsp;</b><br>&nbsp;</span></p>)
<p style="line-height:20% " align="left"><span style="font-size:0.9em;" ><br><br></span></p>
@box[bg-grey-15 text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.8em;" ><b>&nbsp;</b><br>&nbsp;</span></p>)
@snapend


@snap[north-west span-35 ]
<br>
<br>
<br>
@box[bg-lt-blue-pp text-white rounded my-box-pad2  ](<p style="line-height:70%"><span style="font-size:0.8em;" ><b>Basic Boot Components &lpar;MIN&rpar;</b><br>&nbsp;</span></p>)
@box[bg-lt-blue-pp text-white rounded my-box-pad2  ](<p style="line-height:70%"><span style="font-size:0.8em;" ><b>Advance Boot components&lpar;Full&rpar; </b><br>&nbsp;</span></p>)
@box[bg-lt-blue-pp text-white rounded my-box-pad2  ](<p style="line-height:70%"><span style="font-size:0.8em;" ><b>Close Source</b><br><br>&nbsp;</span></p>)
@snapend



@snap[north-east span-62 fragment]
<br>
<br>

<p style="line-height:30% " align="left"><span style="font-size:0.9em;" ><br><br></span></p>
@css[text-white ](<p style="line-height:60%" align="left" ><span style="font-size:0.7em;" > UEFI, ACPI, PlatformInit<br><br><br><br></span></p>)
@css[text-white ](<p style="line-height:60%" align="left" ><span style="font-size:0.7em;" > SMBIOS, S3, OPAL <br> <br><br><br> </span></p>)
@css[text-white ](<p style="line-height:60%" align="left" ><span style="font-size:0.7em;" > TXT, AMT, CSM <br><br><br><br> </span></p>)

@snapend


@snap[south span-85 fragment]
@box[bg-purple-pp text-white rounded my-box-pad2  ](<p style="line-height:40%"><span style="font-size:0.8em">Add / remove features using a build switch<br><br>&nbsp;</span></p>)
@snapend

Note:

- Minimal set (basic boot component) – this includes the minimal components needed to boot to the UEFI Shell or to a UEFI OS. The feature set is limited and may only include a basic ACPI table and some required platform initialization. 
- Full set (advanced boot component) – this entails all of the components needed to make a production BIOS. For example, it may support S3, SMBIOS table, OPAL. (Storage Work Group Storage Security Subsystem Class: Opal  - i.e. Tpm Opal)
- Most  advanced modules can be open source, too, but there might be a small portion of code that can not be open source, such as the binary elements used by TXT/AMT/CSM 


---?image=assets/images/slides/Slide20.JPG
@title[Basic Boot Components]
<p align="right"><span class="gold" >@size[1.1em](<b>Basic Boot Components</b>)</span><span style="font-size:0.8em;" ><br></span></p>



Note:

Typically all the Intel Architecture platform firmware basic boot components are almost the same. In the slide, the GREEN part means the generic EDK II core modules. 
The YELLOW part means the silicon specific modules. And finally, the PURPLE part means the platform/board specific modules 


In the UEFI scope, we need the variable, timer, CPU, PCI, either SATA or USB as storage, Graphic or terminal as console output, and finally, USB/PS2 Keyboard or terminal as console input. The SMM portion is required for most X86 platforms in order to support UEFI Authenticated Variables [AUTH VARIABLE]. 
Most UEFI OSes also require ACPI, so ACPI tables and an SMM driver to enable/disable ACPI are needed. 
The platform may also need to initiliaze General Purpose Input/Ouput (GPIO) pins or a Super IO (SIO) to enable the basic boot functionality. 



---?image=assets/images/slides/Slide18.JPG
@title[Features Build Enabled]
<p align="right"><span class="gold" >@size[1.1em](<b>Features Build Enabled</b>)</span><span style="font-size:0.8em;" ><br></span></p>

@snap[north-west span-85 ]
<br>
<br>
@box[bg-purple-pp text-white rounded my-box-pad2  ](<p style="line-height:40%"><span style="font-size:0.8em">Platform-Board Build Scripts<br><br>&nbsp;</span></p>)
@snapend


@snap[north-west span-80]
<br>
<br>
<br>
<br>
<p style="line-height:70%" align="left" ><span style="font-size:0.75em;" >
Many platforms have a script (Python or bash) to pre & post process the EDK II build process:  
<a href="https://github.com/tianocore/edk2-platforms/tree/master/Platform/Intel#build">Build Script</a></p>

<p style="line-height:60%" align="left" ><span style="font-size:0.75em;" >
Example: Invoked from the @size[.7em](<font face="Consolas">edk2-platforms/Platform/Intel</font>)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;
          @size[.7em](@color[#A8ff60](<b><font face="Consolas">python build_bios.py –p <Board-name> </font></b>))<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
               @size[.75em]( - uses config file @color[#A8ff60](<font face="Consolas">build.cfg</font>) from the @color[#A8ff60](<font face="Consolas"><Board-name></font>) directory)<br>
			   <br>
</span></p>
@snapend

@snap[south-west span-100]
<p style="line-height:70%" align="left" ><span style="font-size:0.75em;" >
Configuration  Files:
</span></p>
<ul style="list-style-type:disc; line-height:0.6;">
  <li><span style="font-size:0.6em" > <font face="Consolas">edk2-platforms/Platform/Intel/build.cfg </font> - default settings </span> </li>
  <li><span style="font-size:0.6em" > Default settings are under the <font face="Consolas">DEFAULT_CONFIG</font> section</span> </li>
  <li><span style="font-size:0.6em" > Override the <font face="Consolas">edk2-platforms/Platform/Intel/. . ./build.cfg</font> settings from each board in board specific directory</span> </li>
</ul>
<br>
<br>
@snapend


Note:

### Building with the python script
- Open command window, go to the workspace directory, e.g. c:/Kabylake or ~/Kabylake in the case of a linux OS
- If using a linux OS
  - Type "cd edk2"
  - Type "source edksetup.sh"

- Type "cd ../" to go back to the workspace directory
- Type "cd edk2-platforms/Platform/Intel
- Type "python build_bios.py -p REPLACE_WITH_BOARD_NAME"



#### Configuration Files
The edk2-platforms/Platform/Intel/build.cfg file contains the default settings used by build_bios.py

The default settings are under the DEFAULT_CONFIG section

Each board can have a settings file that will override the edk2-platforms/Platform/Intel/build.cfg settings


An example of a board specific settings:
edk2-platforms/Platform/Intel/KabylakeOpenBoardPkg/KabylakeRvp3/build_config.cfg

---?image=assets/images/slides/Slide18.JPG
@title[Example Build Config File]
<p align="right"><span class="gold" >@size[1.1em](<b>Example Build Config File</b>)</span><span style="font-size:0.8em;" ><br></span></p>

@snap[north-west span-80 ]
<br>
<br>
<br>
<br>
<br>
<br>
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend

@snap[north-west span-80 ]
<br>
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" ><br>
Kabylake example of Board specific settings:
</span></p>
<p style="line-height:50% " align="left"><span style="font-size:0.6em; font-family:Consolas;" >
&lt;workspace&gt;/edk2-platforms/\<br>&nbsp;
Platform/Intel/KabylakeOpenBoardPkg/\ <br>&nbsp;&nbsp;
KabylakeRvp3 /  @color[#A8ff60](build_config.cfg) <br></p>

<p style="line-height:40% " align="left"></span><span style="font-size:0.4em; font-family:Consolas;" ><br>&nbsp;&nbsp;
[CONFIG] <br>&nbsp;&nbsp;
WORKSPACE_PLATFORM_BIN = WORKSPACE_PLATFORM_BIN <br>&nbsp;&nbsp;
EDK_SETUP_OPTION = <br>&nbsp;&nbsp;
openssl_path = <br>&nbsp;&nbsp;
PLATFORM_BOARD_PACKAGE = KabylakeOpenBoardPkg <br>&nbsp;&nbsp;
PROJECT = KabylakeOpenBoardPkg/KabylakeRvp3 <br>&nbsp;&nbsp;
BOARD = KabylakeRvp3 <br>&nbsp;&nbsp;
FLASH_MAP_FDF = KabylakeOpenBoardPkg/Include/Fdf/FlashMapInclude.fdf <br>&nbsp;&nbsp;
PROJECT_DSC = KabylakeOpenBoardPkg/KabylakeRvp3/OpenBoardPkg.dsc <br>&nbsp;&nbsp;
BOARD_PKG_PCD_DSC = KabylakeOpenBoardPkg/KabylakeRvp3/OpenBoardPkgPcd.dsc <br>&nbsp;&nbsp;
ADDITIONAL_SCRIPTS = KabylakeOpenBoardPkg/KabylakeRvp3/build_board.py <br>&nbsp;&nbsp;
PrepRELEASE = DEBUG <br>&nbsp;&nbsp;
SILENT_MODE = FALSE <br>&nbsp;&nbsp;
&nbsp;.&nbsp;.&nbsp;.
</span></p>
@snapend


---?image=assets/images/slides/Slide18.JPG
@title[Platform Features Table d’hôte ]
<p align="right"><span class="gold" >@size[1.1em](<b>Platform Features Table d’hôte </b>)</span><span style="font-size:0.8em;" ></span></p>

@snap[south-west span-100 ]
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend

@snap[north-west span-80 ]
<br>
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" ><br>
Platform Firmware Boot Stage PCD in: <br><br><br>
@color[yellow](<b><font face="Consolas">OpenBoardPkgConfig.dsc</font></b>)
</span></p>

@snap[south-west span-100 ]
<p style="line-height:40% " align="left"></span><span style="font-size:0.45em; font-family:Consolas;" ><br>&nbsp;&nbsp;
[PcdsFixedAtBuild]<br>&nbsp;&nbsp;&nbsp;&nbsp;
  &num;<br>&nbsp;&nbsp;&nbsp;&nbsp;
  &num; Please select BootStage here.<br>&nbsp;&nbsp;&nbsp;&nbsp;
  &num; Stage 1 - enable debug (system deadloop after debug init)<br>&nbsp;&nbsp;&nbsp;&nbsp;
  &num; Stage 2 - mem init (system deadloop after mem init)<br>&nbsp;&nbsp;&nbsp;&nbsp;
  &num; Stage 3 - boot to UEFI shell only<br>&nbsp;&nbsp;&nbsp;&nbsp;
  &num; Stage 4 - boot to OS<br>&nbsp;&nbsp;&nbsp;&nbsp;
  &num; Stage 5 - boot to OS with security boot enabled<br>&nbsp;&nbsp;&nbsp;&nbsp;
  &num; Stage 6 – Add Advanced features<br>&nbsp;&nbsp;&nbsp;&nbsp;
  gMinPlatformPkgTokenSpaceGuid.@color[yellow](PcdBootStage)|4
</span></p>
<br>
@snapend


Note:

Features added via table d’hôte  by using a PCD. 
table d’hôte  Pronounced “Tab la dout”

Within the DSC and  FDF choose which modules to include based on PCD

Example
<pre>
DSC:
[PcdsFeatureFlag]
  gMinPlatformPkgTokenSpaceGuid.PcdStopAfterDebugInit|FALSE
  gMinPlatformPkgTokenSpaceGuid.PcdStopAfterMemInit|FALSE
  gMinPlatformPkgTokenSpaceGuid.PcdBootToShellOnly|FALSE
  gMinPlatformPkgTokenSpaceGuid.PcdUefiSecureBootEnable|FALSE
  gMinPlatformPkgTokenSpaceGuid.PcdTpm2Enable|FALSE

!if gMinPlatformPkgTokenSpaceGuid.PcdBootStage >= 1
 gMinPlatformPkgTokenSpaceGuid.PcdStopAfterDebugInit|TRUE
!endif

Example FDF
!if gMinPlatformPkgTokenSpaceGuid.PcdBootToShellOnly == FALSE
INF  MdeModulePkg/Universal/FaultTolerantWriteDxe/FaultTolerantWriteSmm.inf
INF  MdeModulePkg/Universal/Variable/RuntimeDxe/VariableSmmRuntimeDxe.inf
INF  MdeModulePkg/Universal/Variable/RuntimeDxe/VariableSmm.inf
!endif
</pre>


Depending on the stage # provides some idea regarding what components are needed for a BIOS solution. It can be 3M full featured BIOS, or only 256K if just the basic boot is required, in some cases. 

This work can be done by defining some default configuration in PlatformConfig.dsc. 
For example, PcdBootStage|4 can be used to configure a BIOS to support a boot to OS (with ACPI/SMM), or PcdBootStage|3 to configure a BIOS to boot to shell only (without ACPI/SMM) 

- Stage I - Minimal Debug
  - Serial Port, Port 80, External debuggers Optional: Software debugger
- Stage II  - Memory Functional
  - Basic hardware initialization including main memory
- Stage III - Boot to UEFI Shell
   - Generic DXE driver execution
- Stage IV - Boot to OS
  - Boot a general purpose operating system with the minimally required feature set. Publish a minimal set of ACPI tables.- Stage V -Security Enabled
  - UEFI Secure Boot, TCG trusted boot, DMA protection, etc.
- Stage VI - Advanced Feature Selection
  - Firmware update, power management, networking support, manageability, testability, reliability, availability, serviceability, non-essential provisioning and resiliency mechanisms
- Stage VII – Tuning
   - Size and performance optimizations


---?image=assets/images/slides/Slide18.JPG
@title[Platform Features à la carte with PCDs ]
<p align="right"><span class="gold" >@size[1.1em](<b>Platform Features à la carte with PCDs  </b>)</span><span style="font-size:0.8em;" ></span></p>

@snap[south-west span-100 ]
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend

@snap[north-west span-80 ]
<br>
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" ><br>
The Platform Config <b>.<font face="Consolas">dsc</font></b> file controls if feature ON or OFF
<br><br>
Example:
<br>
@color[yellow](<b><font face="Consolas">OpenBoardPkgConfig.dsc</font></b> -  à la carte)
</span></p>

@snap[south-west span-100 ]
<p style="line-height:40% " align="left"></span><span style="font-size:0.45em; font-family:Consolas;" ><br>&nbsp;&nbsp;
[PcdsFeatureFlag]<br>&nbsp;&nbsp;&nbsp;&nbsp;
  gMinPlatformPkgTokenSpaceGuid.@color[yellow](PcdStopAfterDebugInit)|FALSE <br>&nbsp;&nbsp;&nbsp;&nbsp;
  gMinPlatformPkgTokenSpaceGuid.@color[yellow](PcdStopAfterMemInit)|FALSE <br>&nbsp;&nbsp;&nbsp;&nbsp;
  gMinPlatformPkgTokenSpaceGuid.@color[yellow](PcdBootToShellOnly)|FALSE <br>&nbsp;&nbsp;&nbsp;&nbsp;
  gMinPlatformPkgTokenSpaceGuid.@color[yellow](PcdUefiSecureBootEnable)|FALSE <br>&nbsp;&nbsp;&nbsp;&nbsp;
  gMinPlatformPkgTokenSpaceGuid.@color[yellow](PcdTpm2Enable)|FALSE  <br>&nbsp;&nbsp;&nbsp;&nbsp;
!if gMinPlatformPkgTokenSpaceGuid.PcdBootStage >= 1 <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  gMinPlatformPkgTokenSpaceGuid.@color[yellow](PcdStopAfterDebugInit)|TRUE <br>&nbsp;&nbsp;&nbsp;&nbsp;
!endif <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
 . &nbsp;&nbsp;.&nbsp;&nbsp; .

</span></p>
<br>
@snapend


Note:

At the same time, a platform firmware may provide an “à la carte” menu so that an advanced user can configure an individual item. For example, PcdUefiSecureBootEnable can be used to configure if a BIOS needs to support UEFI secure boot [AUTH VARIABLE]. PcdTpm2Enable can be used to configure if a BIOS needs to support the TPM2 [TPM2 EDKII]. 


---
@title[Where are the DSC & FDF files? ]
<p align="right"><span class="gold" >@size[1.1em](<b>Where are the DSC & FDF files?</b>)</span><span style="font-size:0.8em;" ></span></p>

@snap[north-west span-48 ]
<br>
<br>
<br>
<br>

@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend

@snap[north-east span-48 ]
<br>
<br>
<br>
<br>

@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend

@snap[north-west span-45 ]
<br>
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" ><br>
Kabylake Open Board
</span></p>
<p style="line-height:40% " align="left"></span><span style="font-size:0.45em; font-family:Consolas;" ><br>&nbsp;&nbsp;
Platform/Intel /<br>&nbsp;&nbsp;&nbsp;
 KabyLakeOpenBoardPkg /<br>&nbsp;&nbsp;&nbsp;&nbsp;
 
  KabyLakeRvp3 /<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     OpenBoardPkg@color[yellow](Config).dsc <br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

     OpenBoardPkgPcd.dsc  <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     OpenBoardPkgBuildOption.dsc<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     OpenBoardPkg.dsc<br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

     FlashMapInclude.fdf  <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     OpenBoardPkg.fdf 

</span></p>

@snapend


@snap[north-east span-47 ]
<br>
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" ><br>
MinnowBoard Turbot
</span></p>

<p style="line-height:40% " align="left"><span style="font-size:0.45em; font-family:Consolas;" ><br>&nbsp;&nbsp;
Platform/Intel /<br>&nbsp;&nbsp;&nbsp;&nbsp;

  Vlv2TbltDevicePkg /<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     PlatformPkg.dec<br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

     PlatformPkg@color[yellow](Config).dsc<br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

     PlatformPkgIa32.dsc<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     PlatformPkgX64.dsc<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     PlatformPkgGcc.dsc<br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

     PlatformPkg.fdf <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     PlatformPkgGcc.fdf

</span></p>

@snapend

@snap[south span-95 fragment]
@box[bg-purple-pp text-white rounded my-box-pad2  ](<p style="line-height:40%"><span style="font-size:0.8em">..Config.dsc File Controls if feature ON or OFF <br><br>&nbsp;</span></p>)
@snapend


Note:

---
@title[Example Kabylake Configuration .DSC file ]
<p align="right"><span class="gold" >@size[1.1em](<b>Example Kabylake Configuration .DSC file</b>)</span><span style="font-size:0.8em;" ></span></p>

```
[PcdsFixedAtBuild]
  #
  # Please select BootStage here.
  # Stage 1 - enable debug (system deadloop after debug init)
  # Stage 2 - mem init (system deadloop after mem init)
  # Stage 3 - boot to shell only
  # Stage 4 - boot to OS
  # Stage 5 - boot to OS with security boot enabled
  #
  gMinPlatformPkgTokenSpaceGuid.PcdBootStage|4
  
[PcdsFeatureFlag]
  gMinPlatformPkgTokenSpaceGuid.PcdStopAfterDebugInit|FALSE
  gMinPlatformPkgTokenSpaceGuid.PcdStopAfterMemInit|FALSE
  gMinPlatformPkgTokenSpaceGuid.PcdBootToShellOnly|FALSE
  gMinPlatformPkgTokenSpaceGuid.PcdUefiSecureBootEnable|FALSE
  gMinPlatformPkgTokenSpaceGuid.PcdTpm2Enable|FALSE

!if gMinPlatformPkgTokenSpaceGuid.PcdBootStage >= 1
  gMinPlatformPkgTokenSpaceGuid.PcdStopAfterDebugInit|TRUE
!endif

!if gMinPlatformPkgTokenSpaceGuid.PcdBootStage >= 2
  gMinPlatformPkgTokenSpaceGuid.PcdStopAfterDebugInit|FALSE
  gMinPlatformPkgTokenSpaceGuid.PcdStopAfterMemInit|TRUE
!endif
!if gMinPlatformPkgTokenSpaceGuid.PcdBootStage >= 3
  gMinPlatformPkgTokenSpaceGuid.PcdStopAfterMemInit|FALSE
  gMinPlatformPkgTokenSpaceGuid.PcdBootToShellOnly|TRUE
!endif

!if gMinPlatformPkgTokenSpaceGuid.PcdBootStage >= 4
  gMinPlatformPkgTokenSpaceGuid.PcdBootToShellOnly|FALSE
!endif

!if gMinPlatformPkgTokenSpaceGuid.PcdBootStage >= 5
  gMinPlatformPkgTokenSpaceGuid.PcdUefiSecureBootEnable|TRUE
  gMinPlatformPkgTokenSpaceGuid.PcdTpm2Enable|TRUE
!endif
  
  gBoardModuleTokenSpaceGuid.PcdTbtEnable|FALSE
 
 
  gBoardModuleTokenSpaceGuid.PcdMultiBoardSupport|TRUE
 

```


@snap[south-east span-45 ]
<p style="line-height:40% " align="left"><span style="font-size:0.55em;">
Link to <a href="https://github.com/tianocore/edk2-platforms/blob/master/Platform/Intel/KabylakeOpenBoardPkg/KabylakeRvp3/OpenBoardPkgConfig.dsc">Confg .dsc </a>
file
</span></p>
@snapend

Note:

Click on link to view the whole .DSC file

---
@title[Example Kabylake Configuration .FDF file ]
<p align="right"><span class="gold" >@size[1.1em](<b>Example Kabylake Configuration .FDF file</b>)</span><span style="font-size:0.8em;" ></span></p>

```
[FD.KabylakeRvp3]
 #
  . . .
[FV.FvPreMemory]
BlockSize          = $(FLASH_BLOCK_SIZE)
FvAlignment        = 16
  . . .
INF  UefiCpuPkg/SecCore/SecCore.inf
INF  MdeModulePkg/Core/Pei/PeiMain.inf
!include $(PLATFORM_PACKAGE)/Include/Fdf/CorePreMemoryInclude.fdf

INF $(PLATFORM_PACKAGE)/PlatformInit/ReportFv/ReportFvPei.inf
INF $(PLATFORM_PACKAGE)/PlatformInit/PlatformInitPei/PlatformInitPreMem.inf
INF IntelFsp2WrapperPkg/FspmWrapperPeim/FspmWrapperPeim.inf
INF $(PLATFORM_PACKAGE)/PlatformInit/SiliconPolicyPei/SiliconPolicyPeiPreMem.inf

. . .

[FV.FvPostMemoryUncompact]
BlockSize          = $(FLASH_BLOCK_SIZE)
FvAlignment        = 16
ERASE_POLARITY     = 1
 . . .

!include $(PLATFORM_PACKAGE)/Include/Fdf/CorePostMemoryInclude.fdf

# Init Board Config PCD
INF $(PLATFORM_PACKAGE)/PlatformInit/PlatformInitPei/PlatformInitPostMem.inf
INF IntelFsp2WrapperPkg/FspsWrapperPeim/FspsWrapperPeim.inf
INF $(PLATFORM_PACKAGE)/PlatformInit/SiliconPolicyPei/SiliconPolicyPeiPostMem.inf

  . . .

[FV.FvUefiBootUncompact]
BlockSize          = $(FLASH_BLOCK_SIZE)
FvAlignment        = 16
 . . .

!include $(PLATFORM_PACKAGE)/Include/Fdf/CoreUefiBootInclude.fdf

INF  UefiCpuPkg/CpuDxe/CpuDxe.inf
INF  MdeModulePkg/Bus/Pci/PciHostBridgeDxe/PciHostBridgeDxe.inf

INF  MdeModulePkg/Bus/Pci/SataControllerDxe/SataControllerDxe.inf
INF  MdeModulePkg/Bus/Ata/AtaBusDxe/AtaBusDxe.inf
INF  MdeModulePkg/Bus/Ata/AtaAtapiPassThru/AtaAtapiPassThru.inf
INF  MdeModulePkg/Universal/Console/GraphicsOutputDxe/GraphicsOutputDxe.inf

INF  ShellPkg/Application/Shell/Shell.inf

INF  $(PLATFORM_PACKAGE)/PlatformInit/PlatformInitDxe/PlatformInitDxe.inf
INF  IntelFsp2WrapperPkg/FspWrapperNotifyDxe/FspWrapperNotifyDxe.inf

INF  $(PLATFORM_PACKAGE)/Test/TestPointStubDxe/TestPointStubDxe.inf



```


@snap[south-east span-45 ]
<p style="line-height:40% " align="left"><span style="font-size:0.55em;">
Link to <a href="https://github.com/tianocore/edk2-platforms/blob/master/Platform/Intel/KabylakeOpenBoardPkg/KabylakeRvp3/OpenBoardPkg.fdf"> Kabylake .FDF </a>
file
</span></p>
@snapend

Note:

Click on the link to view the whole .FDF file


---?image=assets/images/slides/Slide28.JPG
@title[Focus - Configuration Section]
<br>
<p align="left"><span class="gold" >@size[1.1em](<b>Configuration </b>)</span></span></p>

@snap[south-west span-30 ]
<br>
<ul style="list-style-type:disc; line-height:0.7;">
  <li><span style="font-size:0.65em" >Setup Variable</span> </li>
  <li><span style="font-size:0.65em" >PCD </span> </li>
  <li><span style="font-size:0.65em" >Policy Hob/PPI/Protocol </span> </li>
</ul>
<br>
<br>
<br>
@snapend


Note:
Configuration. From which interface can a platform module get the configuration data? 

there might be many sources of platform configuration data 

- Setup Variable
- PCD
- Policy Hob/PPI/Protocol

---
@title[Configuration Options ]
<p align="right"><span class="gold" >@size[1.1em](<b>Configuration Options</b>)</span><span style="font-size:0.8em;" ></span></p>

<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" >
There might be many sources of platform configuration data.  </span></p>

@snap[north-west span-30 ]
<br>
<br>
<br>
<br>
@box[bg-royal text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" >PI PCD <br><br>&nbsp;</span></p>)
@box[bg-royal text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" >UEFI Variable<br><br>&nbsp;</span></p>)
@box[bg-royal text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" >FSP UPD <br><br>&nbsp;</span></p>)
@snapend


@snap[north span-30 ]
<br>
<br>
<br>
<br>
@box[bg-royal text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" >Silicon Policy Hob/PPI/Protocol <br>&nbsp;</span></p>)
@box[bg-royal text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" >Configuration Block <br><br>&nbsp;</span></p>)
@box[bg-royal text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" >Global NVS <br><br>&nbsp;</span></p>)
@snapend


@snap[north-east span-30 ]
<br>
<br>
<br>
<br>
@box[bg-royal text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" >Platform Signed Data Blob <br>&nbsp;</span></p>)
@box[bg-royal text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" >CMOS <br><br>&nbsp;</span></p>)
@box[bg-royal text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" >MACRO <br><br>&nbsp;</span></p>)
@snapend



Note:
- PI PCD – The PI PCD could be static data fixed at build time or dynamic data updatable at runtime.
- UEFI Variable – The UEFI Variable can be non-volatile data or volatile data, and it is widely used by VFR.
- FSP UPD – FSP UPD can be static default configuration, or a dynamic updatable UPD.
- Silicon Policy Hob/PPI/Protocol – It is policy data constructed at runtime or it can be a hook for silicon code
- Configuration Block – It is a data structure to put all policy data in a block without any C-language data pointer in a policy data
- Global NVS – It is an ACPI region to pass the configuration from the C code to ASL code
- Platform signed data blob – It is read only signed data at build time.
- CMOS – It is simple non-volatile storage, but it is not secure
- MACRO – C-language MACRO. It is fixed at build time.


+++
<!-- .slide: data-background-transition="none" -->
<!-- .slide: data-transition="none" -->
@title[Configuration Options 02 ]
<p align="right"><span class="gold" >@size[1.1em](<b>Configuration Options - details</b>)</span><span style="font-size:0.8em;" ></span></p>
<br>
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" >

@color[yellow](PI PCD) – The PI PCD could be static data fixed at build time or dynamic data updatable at runtime.<br><br>
@color[yellow](UEFI Variable) – The UEFI Variable can be non-volatile data or volatile data, and it is widely used by VFR.<br><br>
@color[yellow](FSP UPD) – FSP UPD can be static default configuration, or a dynamic updatable UPD.<br><br>
@color[yellow](Silicon Policy Hob/PPI/Protocol) – It is policy data constructed at runtime or it can be a hook for silicon code.<br>
</span></p>

Note:

##### PI PCD – The PI PCD could be static data fixed at build time or dynamic data updatable at runtime. 
- PcdsFeatureFlag: This type PCD only supports 1/0. Caller uses FeaturePcdGet() to retrieve the value. This type of PCD is mapped to be a MACRO so that a compiler optimization can remove the code scoped by “if(FALSE)”. It is not allowed to set as a PcdsFeatureFlag. 
- PcdsFixedAtBuild: This type of PCD can be mapped to a global variable if the caller uses PcdGet(), or a MACRO if the caller uses FixedPcdGet(). As such, this type of PCD can be used in a data structure definition. It is not allowed to be set as PcdsFixedAtBuild. 
- PcdsPatchableInModule: This type of PCD is mapped to a global variable. It is allowed for use by both PcdGet and PcdSet. If PcdSet is called, it only changes the module-level PCD value instead of a system-level PCD value. Only the current module sees the PCD change. Other modules still see the original value. 
- PcdsDynamicDefault: PcdsDynamicDefault is mapped to a PPI or protocol. It is allowed for both PcdGet and PcdSet. PcdSet changes the system-level PCD value immediately. This type of PCD value is volatile. The changed value will not be saved in the next boot. 
- PcdsDynamicHii: PcdsDynamicHii is mapped to a UEFI variable. It is non-volatile. As such, the changed value can be saved in the next boot. However, the tricky thing is that this PCD value depends on the UEFI variable services 

readiness. If PcdGet is called before UEFI variable services ready, the default PCD value will be returned instead of the updated PCD value. We suggest that the platform owner be very very careful of this trap. If DXE PcdGet is required before the UEFI variable services are ready, we suggest that the platform define PcdsDynamicDefault, and then use get variable data in the PEI phase to fill in this PCD value. 
- PcdsDynamicVpd: PcdsDynamicVpd is to map configuration data to a static flash region so that a tool can modify the PcdsDynamicVpd after the flash image is generated. This is used by a BIOS that needs to support binary configuration after build. Intel FSP is an example of using PcdsDynamicVpd. 
- PcdsDynamicEx: PcdsDynamicEx is to support external module for binary build. If an EFI module (DXE Driver or PEIM) is not built with the system firmware, the dynamic PCD must be declared as PcdsDynamicEx. If a platform wants to include this binary EFI module, the binary module INF must be included in DSC file. As such, the PCD database will include the external PCD declared in this binary module. 
- SkuIds: SkuIds is a special usage of PCD. The use case of Multi-SKU PCD is to build one UEFI firmware boots on multiple board with different configuration in each board. It can support multiple board configurations generated at build time and support runtime selection to make one configuration take effect finally. The good point is that it is very straightforward for each board, if board configuration can be determined. 
- DefaultStores: DefaultStores is a special usage of PCD (gEfiMdeModulePkgTokenSpaceGuid.PcdSetNvStoreDefaultId). The use case of DefaultStores is to create different default stores in different boot mode, such as standard boot, manufacturing boot mode, or safe boot mode. The configuration data is set to (gEfiMdeModulePkgTokenSpaceGuid.PcdNvStoreDefaultValueBuffer). All those default stores are configured at build time and selected at runtime according to the boot mode. The default store PCD can be consumed by the HiiDatabase to support BIOS setup “load default” operation. 


##### UEFI Variable – The UEFI Variable can be non-volatile data or volatile data, and it is widely used by VFR. 
- In most cases, a non-volatile variable is used to store the user updatable configuration in a setup page. One example is VT enable/disable. This is purely a platform choice. We suggest that the platform map variable configuration to PCD, and use a PcdSet callback to set the variable data. The benefit is that if a new platform just wants to use a static setting, it can remove the variable easily. 
- A non-volatile variable may also be used to store the system configuration generated at runtime, for example, memory configuration data. In order to maintain security, we suggest that platform to lock the configuration variable before exiting PM auth/EndOfDxe event, by using the EDKII_VARIABLE_LOCK protocol. 
- A volatile variable is generated at runtime. A platform setup driver may use this information to control a VFR page to suppress or gray out a menu, or to display the system information, like CPU/SA/PCH stepping and features. 

##### FSP UPD – FSP UPD can be static default configuration, or a dynamic updatable UPD. 
- FSP UPD is used to pass configuration from the FSP wrapper into a FSP binary. A platform needs to convert the policy configuration in PCD to a FSP UPD before calling a FSP API, like FspMemoryInit, FspSiliconInit. 
- PcdsDynamicVpd.Upd: For a FSP binary, we use DynamicVpd.Upd to mark the configuration that needs to be in the UPD region. (Please be aware that UPD is not a standard PCD concept, it is an FSP extension) 

##### Silicon Policy Hob/PPI/Protocol – It is policy data constructed at runtime or it can be a hook for silicon code. 
- Policy data: Silicon Policy Hob/PPI/Protocol are useful in order to let one silicon code module support multiple boards. It is the interface between silicon code and platform code. A platform needs to convert policy configuration in PCD into a Silicon Policy PPI/Protocol. 
- Silicon Hook: Sometimes, we observe that Silicon Policy PPI or Protocol provides a silicon hook for platform. This hook may perform some additional action based on a platform setting, or retrieve some system information. In most cases, we suggest to separate the hook function from policy data. 

+++
<!-- .slide: data-background-transition="none" -->
<!-- .slide: data-transition="none" -->
@title[Configuration Options 03 ]
<p align="right"><span class="gold" >@size[1.1em](<b>Configuration Options - details Cont.</b>)</span><span style="font-size:0.8em;" ></span></p>
<br>
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" >
@color[yellow](Configuration Block) – data structure,  puts all policy data in a block without any C-language data pointer in policy data. <br><br>
@color[yellow](Global NVS) – ACPI region,  passes configuration from C code to ASL code. <br><br>
@color[yellow](Platform signed data blob) – read only, data signed at build time.<br><br>
@color[yellow](CMOS) – simple non-volatile storage, but it is not secure. <br><br>
@color[yellow](MACRO) – C-language MACRO, fixed at build time.<br>
</span></p>

Note:

##### Configuration Block – It is a data structure to put all policy data in a block without any C-language data pointer in a policy data. 
- The Configuration Block is a new idea to resolve data pointer issues objserved in earlier HOB usage. Previously, a silicon code module would define a root policy data object with some data pointers to sub-regions. For example, a PCH policy data may include a pointer to USB policy data, a pointer to SATA policy data, and a pointer to PCIExpress policy data. This HOB policy data pointer needs to be relocated after memory initialization, such as the Memory Reference Code (MRC), is done, because the PEI core needs to move data from temporary memory or Cache-As-RAM (CAR) to DRAM. If the platform code forgets to move the policy data and fix the pointer, the following code might retrieve the wrong policy data pointer. With the configuration block, the silicon and platform needn’t worry about the invalid policy data pointer issue because the data pointer is eliminated. The PCH policy data block may include a USB policy data block, a SATA policy data and a PCIExpress policy data. Configuration Blocks can be mapped and used by either Hob/PPI/Protocol or PCD. 
##### Global NVS – It is an ACPI region to pass the configuration from the C code to ASL code. 
- Global NVS can be used for turning some feature on/off. An example includes returning different _STA values. 0x0 means the device does not exist. 0xF means the device exists. 

It can be used as the policy data, for example CriticalTemperature value. A platform C code module may convert the configuration from PCD to Global NVS. 
- Since the name has “global”, we observe that may platform put all different features into one big data structure. It is discouraged. We recommend each separate feature can have its own NVS data structure. It is easy for feature on/off control. 

##### Platform signed data blob – It is read only signed data at build time. 

This signed data blob provides the configuration on a platform. An OEM may update the configuration for different boards. We suggest the platform map the signed data blob to PCDs so that a platform consumer can just use PcdGet to get the configuration without knowing the data source. The benefit is that all the platform code can be consistent, irrespective of whether the configuration data is from a signed data blob, a BIOS boot block static region, or a UEFI variable. 
##### CMOS – It is simple non-volatile storage, but it is not secure. 
- The most useful usage of CMOS is to use CMOS-CLEAR to determine if an end user wants to use the default variable configuration. This is a consistent user experience from old legacy BIOS. The new platform can use a special function key or a special GPIO as indicator of this logic. 
- The benefit of CMOS is that the CMOS can be accessed at early SEC phase without rich API requirements. Beyond that usage, though, we do not suggest a platform use CMOS to store configuration data. 
##### MACRO – C-language MACRO. It is fixed at build time. 
- A MACRO can be used as static data configuration. It is useful if the MACRO is only used in one module and does not require user configuration. However, if the MACRO is used across many modules or is configurable, like PCIE_BASE, we suggest using PCD. 
- A MACRO used in #IFDEF can be used to enable/disable features. If the consumer is in C code, it can be replaced by FeaturePcd. It will become if(0) or if(1) finally. The benefit of using PCD is that all the code in both path can be built. 


---
@title[TIP: Use PCD Instead of UEFI Variable  ]
<p align="right"><span class="gold" >@size[1.1em](<b>TIP: Use PCD Instead of UEFI Variable </b>)</span><span style="font-size:0.8em;" ></span></p>

@snap[north-west span-48 ]
<br>
<br>
<br>
<br>

@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend

@snap[north-east span-48 ]
<br>
<br>
<br>
<br>

@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend

@snap[north-west span-45 ]
<br>
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" ><br>
@color[cyan](UEFI Variable)
</span></p>

<p style="line-height:40% " align="left"></span><span style="font-size:0.45em; font-family:Consolas;" ><br>&nbsp;&nbsp;
 //<br>&nbsp;&nbsp;
 // Get config from setup variable<br>&nbsp;&nbsp;
 //<br>&nbsp;&nbsp;
 VarDataSize = sizeof (SETUP_DATA);<br>&nbsp;&nbsp;
 Status = GetVariable (<br>&nbsp;&nbsp;&nbsp;&nbsp;
	L"Setup",<br>&nbsp;&nbsp;&nbsp;&nbsp;
	&amp;gSetupVariableGuid,<br>&nbsp;&nbsp;&nbsp;&nbsp;
	NULL,<br>&nbsp;&nbsp;&nbsp;&nbsp;
	&amp;VarDataSize,<br>&nbsp;&nbsp;&nbsp;&nbsp;
	&amp;mSystemConfiguration<br>&nbsp;&nbsp;
 );<br>&nbsp;&nbsp;
</span></p> 
@snapend


@snap[north-east span-47 ]
<br>
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" ><br>
@color[cyan](PCD)
</span></p>

<p style="line-height:40% " align="left"></span><span style="font-size:0.45em; font-family:Consolas;" ><br>&nbsp;&nbsp;
 //<br>&nbsp;&nbsp;
 // Get setup configuration from PCD<br>&nbsp;&nbsp;
 //<br>&nbsp;&nbsp;
 CopyMem (<br>&nbsp;&nbsp;&nbsp;&nbsp;
	&amp;mSystemConfiguration,<br>&nbsp;&nbsp;&nbsp;&nbsp;
	PcdGetPtr (PcdSetupConfiguration),<br>&nbsp;&nbsp;&nbsp;&nbsp;
	sizeof(mSystemConfiguration)<br>&nbsp;&nbsp;&nbsp;&nbsp;
 );
</span></p> 
@snapend


Note:

IF UEFI Variable is used, people then have to remember clearly on which configuration is stored in which direct location. Also if a platform decides to change the configuration source from one place (UEFI variable) to another (static region in boot block), all impacted platform modules are required to change. This is non-ideal for source code maintenance and development. 

BKM is using PCDs only in platform code, no matter where a platform chooses to store configuration data based upon the production requirement, 


Then the code is consistent and easy to maintain, especially if the next generation platform decides to change the location. 

---?image=/assets/images/slides/Slide33.JPG
@title[PCD Syntax review]
<p align="right"><span class="gold" ><b>PCD Syntax Review</b></span></p>
@snap[north-east span-90 fragment]
<br>
<br>
<p align="left" style="line-height:80%"><span style="font-size:0.9em; ">PCD defined in the DEC file from any package</span></p>
@box[bg-black text-white my-box-pad2  ](<p style="line-height:40%" align="left"><span style="font-size:0.450em; font-family:Consolas; " >&nbsp;&nbsp;[Guids.common]<br>&nbsp;&nbsp;@color[red](PcdTokenSpaceGuidName)={ 0xXXXXXXXX, 0xXXXX, 0xXXXX, { 0xXX, . . .}}<br>&nbsp;&nbsp;. . .<br>&nbsp;&nbsp;[Pcds...]<br>&nbsp;&nbsp;PcdTokenSpaceGuidName.PcdTokenName|Value[|DatumType[|MaxSize]]|Token<br>&nbsp;&nbsp;</span></p>)
<br>
@snapend

@snap[north-east span-90 fragment]
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<p align="left" style="line-height:80%"><span style="font-size:0.9em; ">PCD usage listed in INF file for module</span></p>
@box[bg-black text-white my-box-pad2  ](<p style="line-height:40%" align="left"><span style="font-size:0.450em; font-family:Consolas; " >&nbsp;&nbsp;[...Pcds...]<br>&nbsp;&nbsp;PcdTokenSpaceGuidName.@color[red](PcdTokenName)|[Value]<br>&nbsp;&nbsp;</span></p>)
<br>
@snapend


@snap[north-east span-90 fragment]
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<p align="left" style="line-height:80%"><span style="font-size:0.9em; ">Value of PCD set in @color[yellow](<b><font face="Consolas">OpenBoardPkg. . .dsc</font></b>)</span></p>
@box[bg-black text-white my-box-pad2  ](<p style="line-height:40%" align="left"><span style="font-size:0.450em; font-family:Consolas; " >&nbsp;&nbsp;[Pcds...]<br>&nbsp;&nbsp;PcdTokenSpaceGuidName.@color[red](PcdTokenName)|@color[cyan](Value)[|DatumType[|MaximumDatumSize]]</span><br>&nbsp;&nbsp;</p>)
<br>
@snapend



Note:

BKM is to set the desired value in the Platform Specific .DSC file


- PCD defined in the DEC file from any package
- [Guids.common]
   - PcdTokenSpaceGuidName={ 0x914AEBE7, 0x4635, 0x459b, { 0xAA, . . .}}

- [Pcds...]
  - PcdTokenSpaceGuidName.PcdTokenName|Value[|DatumType[|MaxSize]]|Token
  - PCD usage listed in INF file for module
- […Pcd…] 
  - PcdTokenSpaceGuidName.PcdTokenName|[Value]
  - Value of PCD set in NewPlatform.dsc
- [Pcds...]
  - PcdTokenSpaceGuidName.PcdTokenName|Value[|DatumType[|MaximumDatumSize]]
- Example used in new OpenBoardPkg.dsc
- [PcdsFixedAtBuild.IA32]
  -  gEfiIchTokenSpaceGuid.PcdIchAcpiIoPortBaseAddress|0x400
  -  gNewProjectTokenSpaceGuid.PcdFlashAreaBaseAddress|0xFFF00000
  - gNewProjectTokenSpaceGuid.PcdFlashAreaSize|0x100000


 
---?image=/assets/images/slides/Slide34.JPG
@title[How to Map PCD ]
<p align="right"><span class="gold" >@size[1.1em](<b>How to Map PCD to Configuration Data</b>)</span><span style="font-size:0.8em;" ></span></p>
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" >
Using "@color[yellow](Callback)" mechanism to convert PCD to Configuration data
</span></p>
<p style="line-height:70%" align="left" ><span style="font-size:0.75em;" >
No examples currently available
</span></p>



Note:

A PCD driver can provide a callback function on PcdSet(). 
A platform may introduce a “ConfigConvert” module (GREEN box). The logic runs early and calls PcdSet() to convert other storage data (variable, signed data blob, policy hob) to the PCD database. 

Then the rest of PlatformInit code (YELLOW box) can just call PcdGet() to get the policy data. 

If the Platform driver wants to update a PCD value by calling PcdSet() later, the “ConfigConvert” can register a PCD callback function to redirect setting to other source (for example, variable). 

The Key Point is that the purpose of the configuration conversion is that any other platform driver should use PcdGet() to retrieve policy data, and PcdSet() to update policy data. 

KabylakeOpenBoardPkg does not use a UEFI variable to save the configuration data, but this might be used for other real platforms. 


---
@title["C" Data Structure as PCDs ]
<p align="right"><span class="gold" >@size[1.1em](<b>"C" Data Structure as PCDs</b>)</span><span style="font-size:0.8em;" ></span></p>

@snap[north-west span-100 ]
<br>
<br>
<br>
<br>
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend


@snap[north-west span-100 ]
<br>
<p style="line-height:70%" align="left" ><span style="font-size:0.75em;" ><br>
Example: <b><font face="Consolas">AdvancedFeaturePkg.dec</font></b>  for SMBIOS type 0 data structure
</span></p>

<p style="line-height:35% " align="left"></span><span style="font-size:0.4em; font-family:Consolas;" ><br>&nbsp;&nbsp;
gAdvancedFeaturePkgTokenSpaceGuid.PcdSmbiosType0BiosInformation| \<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
        {0x0}|@color[yellow](SMBIOS_TABLE_TYPE0)|0x80010000 {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    &lt;HeaderFiles&gt;<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
       IndustryStandard/@color[yellow](SmBios.h)<br>&nbsp;&nbsp;&nbsp;&nbsp;
    &lt;Packages&gt;<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
       MdePkg/MdePkg.dec<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
       AdvancedFeaturePkg/AdvancedFeaturePkg.dec<br>&nbsp;&nbsp;
}<br>&nbsp;&nbsp;
gAdvancedFeaturePkgTokenSpaceGuid.PcdSmbiosType0BiosInformation.Vendor|0x1<br>&nbsp;&nbsp;
gAdvancedFeaturePkgTokenSpaceGuid.PcdSmbiosType0BiosInformation.BiosVersion|0x2<br>&nbsp;&nbsp;
gAdvancedFeaturePkgTokenSpaceGuid.PcdSmbiosType0BiosInformation.BiosSegment|0xF000<br>&nbsp;&nbsp;
gAdvancedFeaturePkgTokenSpaceGuid.PcdSmbiosType0BiosInformation.BiosReleaseDate|0x3<br>&nbsp;&nbsp;
gAdvancedFeaturePkgTokenSpaceGuid.PcdSmbiosType0BiosInformation.BiosSize|0xFF<br>&nbsp;&nbsp;
gAdvancedFeaturePkgTokenSpaceGuid.PcdSmbiosType0BiosInformation.BiosCharacteristics.\<br>&nbsp;&nbsp;&nbsp;&nbsp;
     PciIsSupported|1<br>&nbsp;&nbsp;
gAdvancedFeaturePkgTokenSpaceGuid.PcdSmbiosType0BiosInformation.BiosCharacteristics.\<br>&nbsp;&nbsp;&nbsp;&nbsp;
     PlugAndPlayIsSupported|1
</span></p>




Note:

In the latest EDKII, the developer can associate a PCD to a C data structure and assign the value to each sub-field directly. 

MinPlatform AdvancedFeaturePkg  .DEC file defines a set of PCD for SMBIOS table data structure. Each field has its own default value. 

By using the structure PCD, a platform may define a global configuration PCD and assign the policy data in DSC file directly. 

From SmBios.h 
<pre>
typedef struct {
  SMBIOS_STRUCTURE          Hdr;
  SMBIOS_TABLE_STRING       Vendor;
  SMBIOS_TABLE_STRING       BiosVersion;
  UINT16                    BiosSegment;
  SMBIOS_TABLE_STRING       BiosReleaseDate;
  UINT8                     BiosSize;
  MISC_BIOS_CHARACTERISTICS BiosCharacteristics;
  UINT8                     BIOSCharacteristicsExtensionBytes[2];
  UINT8                     SystemBiosMajorRelease;
  UINT8                     SystemBiosMinorRelease;
  UINT8                     EmbeddedControllerFirmwareMajorRelease;
  UINT8                     EmbeddedControllerFirmwareMinorRelease;
  //
  // Add for smbios 3.1.0
  //
  EXTENDED_BIOS_ROM_SIZE    ExtendedBiosSize;
} SMBIOS_TABLE_TYPE0;

</pre>

---
@title[Example of DSC xRef .DEC & .h  files  ]
<p align="right"><span class="gold" >@size[1.1em](<b>Example of DSC xRef &lpar;DEC & .h &rpar; </b>)</span><span style="font-size:0.8em;" ></span></p>

@snap[north-west span-48 ]
<br>
<br>
<br>
<br>

@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend

@snap[north-east span-48 ]
<br>
<br>
<br>
<br>

@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend

@snap[north-west span-47 ]
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" ><br><br>
@color[cyan](Purley Pkg  DEC File)<br>
@size[.8em](&num;&num; <font face="Consolas">gEfiSetupVariableGuid</font>)
</span></p>

<p style="line-height:35% " align="left"></span><span style="font-size:0.4em; font-family:Consolas;" >&nbsp;&nbsp;
 OemSkuTokenSpaceGuid.PcdSetupData|{0x0}|\<br>&nbsp;&nbsp;
 @color[yellow](SYSTEM_CONFIGURATION)|0x000F0001 &lbrace; <br>&nbsp;&nbsp;
    &lt;HeaderFiles&gt; <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
      Guid/@color[yellow](SetupVariable.h)  <br>&nbsp;&nbsp;&nbsp;&nbsp;
    &lt;Packages&gt;<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
      MdePkg/MdePkg.dec<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
      PurleyRcPkg/RcPkg.dec<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
      PurleySktPkg/SocketPkg.dec<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
      LewisburgPkg/PchRcPkg.dec<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
      PurleyOpenBoardPkg/PlatPkg.dec<br>&nbsp;&nbsp;
  &rbrace;<br><br>
<font face="Arial">@size[1.5em](@color[cyan](&nbsp;&nbsp;"C" SetupVariable.h File  )) </font>
<br>
&nbsp;&nbsp;.&nbsp; . &nbsp;.<br>&nbsp;&nbsp;&nbsp;&nbsp;
   UINT8    FanPwmOffset; <br>&nbsp;&nbsp;&nbsp;&nbsp;
   UINT8    WakeOnLanSupport;<br>&nbsp;&nbsp;&nbsp;&nbsp;
   UINT8    Use1GPageTable;<br>&nbsp;&nbsp;&nbsp;&nbsp;
   UINT8    CloudProfile;<br>&nbsp;&nbsp;
} @color[yellow](SYSTEM_CONFIGURATION);
</span></p> 
@snapend


@snap[north-east span-47 ]
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" ><br><br>
@color[cyan](StructureConfig.DSC File)
</span></p>
<br>
<p style="line-height:35% " align="left"></span><span style="font-size:0.4em; font-family:Consolas;" ><br>&nbsp;&nbsp;
gOemSkuTokenSpaceGuid.PcdSetupData.\<br>&nbsp;&nbsp;
@color[#ff0000](CloudProfile)|0x0 <br>
<br>&nbsp;&nbsp;
gOemSkuTokenSpaceGuid.PcdSetupData.\<br>&nbsp;&nbsp;
@color[#ff0000](Use1GPageTable)|0x1<br>
<br>&nbsp;&nbsp;
gOemSkuTokenSpaceGuid.PcdSetupData.\<br>&nbsp;&nbsp;
@color[#ff0000](FanPwmOffset)|0x0 <br>
<br>&nbsp;&nbsp;
gOemSkuTokenSpaceGuid.PcdSetupData.\<br>&nbsp;&nbsp;
@color[#ff0000](WakeOnLanSupport)|0x0
<br>&nbsp;&nbsp;
. &nbsp;. &nbsp;.

</span></p> 
@snapend


Note:

---
@title[Configuration Multi-SKU PCD – Board ID  ]
<p align="right"><span class="gold" >@size[1.1em](<b>Configuration Multi-SKU PCD – Board ID </b>)</span><span style="font-size:0.8em;" ></span></p>

@snap[north-west span-49 ]
<br>
<br>
<br>
<br>

@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend

@snap[north-east span-48 ]
<br>
<br>
<br>
<br>

@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend

@snap[north-west span-48 ]
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" ><br><br>
@color[cyan](DSC File – SKU Set at BUILD time)
</span></p>

<p style="line-height:35% " align="left"></span><span style="font-size:0.4em; font-family:Consolas;" ><br>&nbsp;&nbsp;
 .&nbsp;.&nbsp;.<br>&nbsp;&nbsp;
SKUID_IDENTIFIER = ?<br>&nbsp;&nbsp;
<br>&nbsp;&nbsp;
[SkuIds]<br>&nbsp;&nbsp;
0|DEFAULT<br>&nbsp;&nbsp;
4|BoardX<br>&nbsp;&nbsp;
0x42|BoardY<br>&nbsp;&nbsp;
<br>&nbsp;&nbsp;
[PcdsDynamicDefault.common.@color[yellow](BoardX)]<br>&nbsp;&nbsp;
gBoardModuleTokenSpaceGuid.PcdGpioPin|0x8<br>&nbsp;&nbsp;
gBoardModuleTokenSpaceGuid.PcdGpioInitValue|\<br>&nbsp;&nbsp;&nbsp;&nbsp;
        &lbrace;0x00, 0x04, 0x02, 0x04, ...&rbrace;<br>&nbsp;&nbsp;
<br>&nbsp;&nbsp;
[PcdsDynamicDefault.common.@color[yellow](BoardY)]<br>&nbsp;&nbsp;
gBoardModuleTokenSpaceGuid.PcdGpioPin|0x4<br>&nbsp;&nbsp;
gBoardModuleTokenSpaceGuid.PcdGpioInitValue|\<br>&nbsp;&nbsp;&nbsp;&nbsp;
        &lbrace;0x00, 0x02, 0x01, 0x02, ...&rbrace;<br>&nbsp;&nbsp;
</span></p> 
@snapend


@snap[north-east span-47 ]
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" ><br><br>
@color[cyan](SKU PCD Set Dynamically)
</span></p>
<br>
<p style="line-height:35% " align="left"></span><span style="font-size:0.4em; font-family:Consolas;" >
&nbsp;&nbsp;
BoardXBoardDetect( VOID)<br>&nbsp;&nbsp;
&lbrace;<br>&nbsp;&nbsp;
. . .<br>&nbsp;&nbsp;&nbsp;&nbsp;
  if (@color[yellow](LibPcdGetSku) () != 0) &lbrace; <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    return EFI_SUCCESS;<br>&nbsp;&nbsp;&nbsp;&nbsp;
  &rbrace; <br>&nbsp;&nbsp;&nbsp;&nbsp;
  if (IsBoardX ()) &lbrace; <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     @color[yellow](LibPcdSetSku) (BoardIdIsBoardX);<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     ASSERT (LibPcdGetSku() == <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
              BoardIdIsBoardX);<br>&nbsp;&nbsp;&nbsp;&nbsp;
  &rbrace;<br>&nbsp;&nbsp;&nbsp;&nbsp;
  return EFI_SUCCESS;<br>&nbsp;&nbsp;
&rbrace;<br>&nbsp;&nbsp;
</span></p> 
@snapend


Note:
SkuIds is a special usage of PCD. It can support multiple configurations generated at build time, and it supports runtime selection to make one configuration take effect finally. 

Multi-sku PCD concept is defined by PI specification Volume 3, Chapter 8 PCD, EFI_PCD_PROTOCOL.SetSku (). 

The SKU PCD is actually a dynamic PCD. During boot, the board detection takes the responsibility to set the SKU. Once the SKU PCD is set, the configuration associated with this SKU takes effect immediately 

---
@title[Default Stores PCD – for Configuration   ]
<p align="right"><span class="gold" >@size[1.1em](<b>Default Stores PCD – for Configuration  </b>)</span><span style="font-size:0.8em;" ></span></p>

@snap[north-west span-100 ]
<br>
<br>
<br>
<br>

@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend



@snap[north-west span-100 ]
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" ><br><br>
@color[cyan](DSC File – )
</span></p>

<p style="line-height:35% " align="left"></span><span style="font-size:0.4em; font-family:Consolas;" ><br>&nbsp;&nbsp;
 .&nbsp;.&nbsp;.<br>&nbsp;&nbsp;
VPD_TOOL_GUID  = 8C3D856A-9 . . .<br>&nbsp;&nbsp;
<br>&nbsp;&nbsp;
[DefaultStores]<br>&nbsp;&nbsp;
0|STANDARD<br>&nbsp;&nbsp;
1|MANUFACTURING<br>&nbsp;&nbsp;
2|SAFE<br>&nbsp;&nbsp;
<br>&nbsp;&nbsp;
<br>&nbsp;&nbsp;
[PcdsDynamicExVpd.common.@color[yellow](DEFAULT)]<br>&nbsp;&nbsp;&nbsp;&nbsp;
  gEfiMdeModulePkgTokenSpaceGuid.PcdNvStoreDefaultValueBuffer|*<br>&nbsp;&nbsp;
[PcdsDynamicEx.common.DEFAULT.@color[yellow](STANDARD)]<br>&nbsp;&nbsp;&nbsp;&nbsp;
  gOemSkuTokenSpaceGuid.PcdSetupData.CloudProfile|0x0<br>&nbsp;&nbsp;&nbsp;&nbsp;
  gOemSkuTokenSpaceGuid.PcdSetupData.Use1GPageTable|0x1<br>&nbsp;&nbsp;
[PcdsDynamicEx.common.DEFAULT.@color[yellow](MANUFACTURING)]<br>&nbsp;&nbsp;&nbsp;&nbsp;
  gOemSkuTokenSpaceGuid.PcdSetupData.CloudProfile|0x1<br>&nbsp;&nbsp;&nbsp;&nbsp;
  gOemSkuTokenSpaceGuid.PcdSetupData.Use1GPageTable|0x0<br>&nbsp;&nbsp;

</span></p> 
@snapend


@snap[north-east span-45 fragment ]
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" ><br><br><br>
&nbsp;
</span></p>
<ul style="list-style-type:disc; line-height:0.7;">
  <li><span style="font-size:0.65em" >Special PCD to support the default stores concept in UEFI specification</span> </li>
  <li><span style="font-size:0.65em" >Can be Dynamically set</span> </li>
</ul>
@snapend


Note:
DefaultStores PCD is a special PCD to support the default stores concept in UEFI specification, Chapter 32 Human Interface Infrastructure. Per UEFI specification, there are 3 standard default stores: Standard Default, Manufacturing Default, and Safe Default 

VPD_TOOL_GUID = 8C3D856A-9BE6-468E-850A-24F7A8D38E08 

#### fdf file 
<pre>
[FD.Platform] 
... 
0x00C50000|0x00030000 
gEfiMdeModulePkgTokenSpaceGuid.PcdVpdBaseAddress 
FILE = $(OUTPUT_DIRECTORY)/$(TARGET)_$(TOOL_CHAIN_TAG)/FV/8C3D856A-9BE6-468E-850A-24F7A8D38E08.bin 
</pre>

Dynampic set in BoardInitPreMem function
PcdSet16S (PcdSetNvStoreDefaultId 
 When its value is set in PEI, it will trig the default setting to be applied as the default EFI variable.

---
@title[Silicon Policy Data Flow Guidelines]
<p align="right"><span class="gold" >@size[1.1em](<b>Silicon Policy Data Flow Guidelines</b>)</span></span></p>

@snap[north-east span-60 ]
<br>
<br>
<br>
@box[bg-grey-15 text-white rounded my-box-pad2  ](<p style="line-height:70%" align="left"><span style="font-size:0.7em;" ><b>&nbsp;</b><br><br>&nbsp;</span></p>)
<p style="line-height:70%" align="left"><br><br>&nbsp; </p>
@box[bg-grey-15 text-white rounded my-box-pad2  ](<p style="line-height:70%" align="left"><span style="font-size:0.7em;" ><b>&nbsp;</b><br><br>&nbsp;</span></p>)
@snapend


@snap[north-west span-45 ]
<br>
<br>
<br>
@box[bg-royal text-white rounded my-box-pad2  ](<p style="line-height:70%"><span style="font-size:0.9em;" ><b>&nbsp;Silicon Module Provides<br>&nbsp; Default Silicon Policy Data</b><br>&nbsp;</span></p>)
<p style="line-height:70%" align="left"><br>&nbsp; </p>
@box[bg-royal text-white rounded my-box-pad2  ](<p style="line-height:70%"><span style="font-size:0.9em;" ><b>&nbsp;Board Module Updates <br>&nbsp;the Silicon Policy Data </b><br><br>&nbsp;</span></p>)
@snapend


@snap[north-east span-50 ]
<br>
<br>
<br>
<p style="line-height:70%" align="left"><span style="font-size:0.7em;" ><b><font face="Consolas">Typedef</font> data structure</b><br><br>&nbsp;</span></p>
<p style="line-height:70%" align="left"><br><br><br>&nbsp; </p>
<p style="line-height:70%" align="left"><span style="font-size:0.7em;" ><b>PCD database, Setup Variable, Binary Blob, etc.</b><br><br>&nbsp;</span></p>
@snapend


Note:

Silicon policy data update is one of the most important tasks as part of bringing up a board. In order to make the board enablement more efficient, we have the below guidelines: 


## Silicon Module Provides Default Silicon Policy Data 

A silicon policy data object is created per Silicon module. The data structure should only contain the data. Functions should not be used in silicon policy data. 
When a silicon module installs this policy data, it should consider the most common usage as the default policy data. 



## Board Module Updates the Silicon Policy Data 

A board module may get default silicon policy data structures and update them. 

A board module may refer to another source to get the board specific policy data, including but not limited to: 
- PCD database 
- Setup Variable 
- Binary Blob 
- Built-in C structure. 

---?image=assets/images/slides/Slide40.JPG
@title[Example: FSP policy in MinPlatformPkg]
<p align="right"><span class="gold" >@size[1.1em](<b>Example: FSP policy in MinPlatformPkg</b>)</span></span></p>


Note:
Take FSP policy in MinPlatformPkg is an example. 

Therein we defined FspPolicyInitLib (FspPolicyInitLib.h) and FspPolicyUpdateLib (FspPolicyUpdateLib.h). 

When the FSP wrapper wants to call MemoryInitAPI, it calls UpdateFspmUpdData() (PeiFspWrapperPlatformLib.c) to get the UPD configuration. 

Then FspmPolicyInit() and FspmPolicyUpdate() are invoked. 

The KabylakeSiliconPkg provides the former (PeiFspPolicyInitLib.c), and KabylakeOpenBoardPkg provides the latter (PeiFspPolicyUpdateLib.c).


---
@title[Update Silicon Policy Example ]
<p align="right"><span class="gold" >@size[1.1em](<b>Update Silicon Policy Example</b>)</span><span style="font-size:0.8em;" ></span></p>

@snap[north-west span-49 ]
<br>
<br>
<br>
<br>
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend

@snap[north-east span-49 ]
<br>
<br>
<br>
<br>
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend


@snap[north-west span-100 ]
<p style="line-height:70%" align="left" ><span style="font-size:0.65em;  font-family:Consolas;" ><br><br>
KabylakeOpenBoardPkg/FspWrapper/Library/PeiSiliconPolicyUpdateLibFsp
</span></p>
@snapend


@snap[north-west span-47 ]
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" ><br><br>&nbsp;
</span></p>

<p style="line-height:35% " align="left"></span><span style="font-size:0.4em; font-family:Consolas;" ><br>&nbsp;&nbsp;
EFI_STATUS<br>&nbsp;&nbsp;
EFIAPI <br>&nbsp;&nbsp;
PeiFspSaPolicyUpdatePreMem ( <br>&nbsp;&nbsp;
IN OUT FSPM_UPD &ast;FspmUpd <br>&nbsp;&nbsp;
) <br>&nbsp;&nbsp;
&lbrace; <br>&nbsp;&nbsp;
VOID &ast;Buffer; <br>&nbsp;&nbsp;
// @color[#A8ff60](<font face="Arial">Override MemorySpdPtr</font>)<br>&nbsp;&nbsp;
CopyMem((VOID &ast;)(UINTN)\  <br>&nbsp;&nbsp;&nbsp;&nbsp;
 FspmUpd-&gt;FspmConfig.MemorySpdPtr00,\ <br>&nbsp;&nbsp;&nbsp;&nbsp;
 (VOID *)(UINTN)PcdGet32(PcdMrcSpdData),\ <br>&nbsp;&nbsp;&nbsp;&nbsp;
 @color[yellow](PcdGet16) (PcdMrcSpdDataSize)); <br>&nbsp;&nbsp;
CopyMem((VOID &ast;)(UINTN)\ <br>&nbsp;&nbsp;&nbsp;&nbsp;
 FspmUpd-&gt;FspmConfig.MemorySpdPtr10,\ <br>&nbsp;&nbsp;&nbsp;&nbsp;
 (VOID &ast;)(UINTN)PcdGet32 (PcdMrcSpdData),\ <br>&nbsp;&nbsp;&nbsp;&nbsp;
 @color[yellow](PcdGet16) (PcdMrcSpdDataSize)); <br>&nbsp;&nbsp;
</span></p> 
@snapend


@snap[north-east span-47 ]
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" ><br><br>&nbsp;
</span></p>

<p style="line-height:35% " align="left"></span><span style="font-size:0.4em; font-family:Consolas;" >
<br>
<br>&nbsp;&nbsp;
@color[#A8ff60](. . .) <br>&nbsp;&nbsp;
// @color[#A8ff60](<font face="Arial">Updating Dq Pins Interleaved,Rcomp</font>) <br>&nbsp;&nbsp;
// @color[#A8ff60](<font face="Arial">Resistor &amp; Rcomp Target Settings </font>)<br>&nbsp;&nbsp;
  <br>&nbsp;&nbsp;&nbsp;&nbsp;
  Buffer = (VOID &ast;) (UINTN) @color[yellow](PcdGet32) \      <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
          (PcdMrcRcompTarget);  <br>&nbsp;&nbsp;&nbsp;&nbsp;
  if (Buffer) &lbrace; <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    CopyMem ((VOID &ast;)\ <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
      FspmUpd-&gt;FspmConfig.RcompTarget, \ <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
      Buffer, 10);  <br>&nbsp;&nbsp;&nbsp;&nbsp;
  &rbrace;  <br>&nbsp;&nbsp;&nbsp;&nbsp;
  return EFI_SUCCESS;  <br>&nbsp;&nbsp;
&rbrace;  
</span></p> 
@snapend


@snap[south-east span-35 ]
<p style="line-height:35% " align="left"></span><span style="font-size:0.4em;" >
Link to file: <a href="https://github.com/tianocore/edk2-platforms/blob/master/Platform/Intel/KabylakeOpenBoardPkg/FspWrapper/Library/PeiSiliconPolicyUpdateLibFsp/PeiSaPolicyUpdatePreMem.c">
PeiSaPolicyUpdatePrMem.c</a>
</span></p> 
@snapend

Note:
One example on how to update silicon policy is shown here


Code is in: 
- KabylakeOpenBoardPkg/FspWrapper/Library/PeiFspPolicyUpdateLib/PeiSaPolicyUpdatePreMem.c

  //
  // If SpdAddressTable are not all 0, it means DIMM slots implemented and
  // MemorySpdPtr* already updated by reading SPD from DIMM in SiliconPolicyInitPreMem.
  //
  // If SpdAddressTable all 0, this is memory down design and hardcoded SpdData
  // should be applied to MemorySpdPtr*.

---
@title[Dynamically Set Defaults ]
<p align="right"><span class="gold" >@size[1.1em](<b>Dynamically Set Defaults</b>)</span><span style="font-size:0.8em;" ></span></p>

@snap[west span-100 ]
<br>
<br>
<br>
<br>
<br>
<br>
<br>
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br>&nbsp;</span></p>)
@snapend

@snap[north-west span-100 ]
<p style="line-height:70%" align="left" ><span style="font-size:0.75em;" ><br><br>
The Default Store PCD is also a dynamic PCD. <br><br>
During boot, the board initialization code checks the boot mode and selects the default store.  <br><br>
This step must be after SetSku. Otherwise, the default setting may be wrong.
</span></p>
@snapend

@snap[west span-100 ]
<br>
<br>
<br>
<br>
<br>
<p style="line-height:35% " align="left"></span><span style="font-size:0.4em; font-family:Consolas;" ><br>&nbsp;
. &nbsp;. &nbsp;.<br>&nbsp;
if (NeedDefaultConfig()) &lbrace; <br>&nbsp;
PcdSet16S (@color[yellow](PcdSetNvStoreDefaultId), 0x0); <br>&nbsp;
&rbrace; <br>&nbsp;
</span></p>
@snapend



Note:

---?image=assets/images/slides/Slide43.JPG
@title[Board Porting ]
<br>
<p align="left"><span class="gold" >@size[1.1em](<b>Board Porting</b>)</span><span style="font-size:0.8em;" ></span></p>

@snap[south-east span-33 ]
<br>
<ul style="list-style-type:disc; line-height:0.7;">
  <li><span style="font-size:0.65em" >GPIO </span> </li>
  <li><span style="font-size:0.65em" >SIO </span> </li>
  <li><span style="font-size:0.65em" >ACPI </span> </li>
  <li><span style="font-size:0.65em" >Stages </span> </li>
</ul>
<br>
<br>
<br>
<br>
@snapend

Note:


Porting. Where are the modules to be ported for a new board? 

Board Specific initialization 



---?image=assets/images/slides/Slide_TableDHote.JPG
@title[Staged Approach by Features]
<p align="right"><span class="gold" >@size[1.1em](<b>Staged Approach by Features</b>)</span><br><span style="font-size:0.75em;" >- Platform Firmware Boot Stage PCD</span></p>
@snap[north-west span-70 ]
<br>
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" >PCD Variable:<br></span>
<span style="font-size:0.5em; font-family:Consolas;">
gPlatformModuleTokenSpaceGuid.PcdBootStage
</span></p>
@snapend

@snap[north-west span-50 ]
<br>
<br>
<br>
<br>
<table id="recTable">
	<tr>
		<td bgcolor="#121212"><p style="line-height:10%"><span style="font-size:0.56em" >Stage 1&nbsp;</span></p></td>
		<td bgcolor="#121212"><p style="line-height:10%"><span style="font-size:0.56em" >Enable Debug &nbsp;</span></p></td>
	</tr>
	<tr>
		<td bgcolor="#323232"><p style="line-height:10%"><span style="font-size:0.56em" >Stage 2&nbsp;</span></p></td>
		<td bgcolor="#323232"><p style="line-height:10%"><span style="font-size:0.56em" >Memory Initialization</span></p></td>
	</tr>
	<tr>
		<td bgcolor="#121212"><p style="line-height:10%"><span style="font-size:0.56em" >Stage 3&nbsp;</span></p></td>
		<td bgcolor="#121212"><p style="line-height:10%"><span style="font-size:0.56em" >Boot to UEFI Shell only &nbsp;</span></p></td>
	</tr>
	<tr>
		<td bgcolor="#323232"><p style="line-height:10%"><span style="font-size:0.56em" >Stage 4&nbsp;</span></p></td>
		<td bgcolor="#323232"><p style="line-height:10%"><span style="font-size:0.56em" >Boot ot OS</span></p></td>
	</tr>
	<tr>
		<td bgcolor="#121212"><p style="line-height:10%"><span style="font-size:0.56em" >Stage 5&nbsp;</span></p></td>
		<td bgcolor="#121212"><p style="line-height:10%"><span style="font-size:0.56em" >Boot ot OS w/ Security enabled&nbsp;</span></p></td>
	</tr>
	<tr>
		<td bgcolor="#323232"><p style="line-height:10%"><span style="font-size:0.56em" >Stage 6&nbsp;</span></p></td>
		<td bgcolor="#323232"><p style="line-height:10%"><span style="font-size:0.56em" >Advanced Feature Selection</span></p></td>
	</tr>
	<tr>
		<td bgcolor="#121212"><p style="line-height:10%"><span style="font-size:0.56em" >Stage 7&nbsp;</span></p></td>
		<td bgcolor="#121212"><p style="line-height:10%"><span style="font-size:0.56em" >Performance Opetimizations &nbsp;</span></p></td>
	</tr>
</table>
<br>
@snapend


@snap[south-east span-45 ]
<p style="line-height:50%" align="left" ><span style="font-size:0.6em;" >
PCD Is tested within .FDF to see which modules to include 
</span></p>
@snapend

Note:
table d’hôte  Pronounced “Tab la dout”
Image source: http://3.bp.blogspot.com/-nCzQh7Xu3_I/Uzk1a4DRk-I/AAAAAAAABCY/lQvT1cbn8Ug/s1600/5892-Caucasian-Man-Sitting-At-A-Table-And-Reading-A-Menu-At-A-Restaurant-Clipart-Illustration.jpg



Depending on the stage # provides some idea regarding what components are needed for a BIOS solution. It can be 3M full featured BIOS, or only 256K if just the basic boot is required, in some cases. 

This work can be done by defining some default configuration in PlatformConfig.dsc. 
For example, PcdBootStage|4 can be used to configure a BIOS to support a boot to OS (with ACPI/SMM), or PcdBootStage|3 to configure a BIOS to boot to shell only (without ACPI/SMM) 

- Stage I - Minimal Debug
  - Serial Port, Port 80, External debuggers Optional: Software debugger
- Stage II  - Memory Functional
  - Basic hardware initialization including main memory
- Stage III - Boot to UEFI Shell
   - Generic DXE driver execution
- Stage IV - Boot to OS
  - Boot a general purpose operating system with the minimally required feature set. Publish a minimal set of ACPI tables.- Stage V -Security Enabled
  - UEFI Secure Boot, TCG trusted boot, DMA protection, etc.
- Stage VI - Advanced Feature Selection
  - Firmware update, power management, networking support, manageability, testability, reliability, availability, serviceability, non-essential provisioning and resiliency mechanisms
- Stage VII – Tuning
   - Size and performance optimizations

Within the DSC and  FDF choose which modules to include based on PCD
Example
<pre>
DSC:
[PcdsFeatureFlag]
  gMinPlatformPkgTokenSpaceGuid.PcdStopAfterDebugInit|FALSE
  gMinPlatformPkgTokenSpaceGuid.PcdStopAfterMemInit|FALSE
  gMinPlatformPkgTokenSpaceGuid.PcdBootToShellOnly|FALSE
  gMinPlatformPkgTokenSpaceGuid.PcdUefiSecureBootEnable|FALSE
  gMinPlatformPkgTokenSpaceGuid.PcdTpm2Enable|FALSE

!if gMinPlatformPkgTokenSpaceGuid.PcdBootStage >= 1
 gMinPlatformPkgTokenSpaceGuid.PcdStopAfterDebugInit|TRUE
!endif

Example FDF
!if gMinPlatformPkgTokenSpaceGuid.PcdBootToShellOnly == FALSE
INF  MdeModulePkg/Universal/FaultTolerantWriteDxe/FaultTolerantWriteSmm.inf
INF  MdeModulePkg/Universal/Variable/RuntimeDxe/VariableSmmRuntimeDxe.inf
INF  MdeModulePkg/Universal/Variable/RuntimeDxe/VariableSmm.inf
!endif
</pre>


---?image=assets/images/slides/Slide45.JPG
<!-- .slide: data-transition="none" -->
@title[Stages vs. Boot Flow]
<p align="right"><span class="gold" >@size[1.1em](<b>Stages vs. Boot Flow</b>)</span><span style="font-size:0.75em;" ></span></p>

Note:

1. enable debug
2. memory initialization
3. boot to UEFI shell only
4. boot to OS
5. boot to OS w/ security enabled
6. Advanced Feature Selection
7. Not shown  Performance Optimizations



+++?image=assets/images/slides/Slide46.JPG
<!-- .slide: data-background-transition="none" -->
<!-- .slide: data-transition="none" -->
@title[Stages vs. Boot Flow 02]
<p align="right"><span class="gold" >@size[1.1em](<b>Stages vs. Boot Flow</b>)</span><span style="font-size:0.75em;" ></span></p>

Note:
1. enable debug
2. memory initialization
3. boot to UEFI shell only
4. boot to OS
5. boot to OS w/ security enabled
6. Advanced Feature Selection
7. Not shown  Performance Optimizations


+++?image=assets/images/slides/Slide47.JPG
<!-- .slide: data-background-transition="none" -->
<!-- .slide: data-transition="none" -->
@title[Stages vs. Boot Flow 03]
<p align="right"><span class="gold" >@size[1.1em](<b>Stages vs. Boot Flow</b>)</span><span style="font-size:0.75em;" ></span></p>

Note:
1. enable debug
2. memory initialization
3. boot to UEFI shell only
4. boot to OS
5. boot to OS w/ security enabled
6. Advanced Feature Selection
7. Not shown  Performance Optimizations


+++?image=assets/images/slides/Slide47_1.JPG
<!-- .slide: data-background-transition="none" -->
<!-- .slide: data-transition="none" -->
@title[Stages vs. Boot Flow 04]
<p align="right"><span class="gold" >@size[1.1em](<b>Stages vs. Boot Flow</b>)</span><span style="font-size:0.75em;" ></span></p>

Note:
1. enable debug
2. memory initialization
3. boot to UEFI shell only
4. boot to OS
5. boot to OS w/ security enabled
6. Advanced Feature Selection
7. Not shown  Performance Optimizations

+++?image=assets/images/slides/Slide48.JPG
<!-- .slide: data-background-transition="none" -->
<!-- .slide: data-transition="none" -->
@title[Stages vs. Boot Flow 05]
<p align="right"><span class="gold" >@size[1.1em](<b>Stages vs. Boot Flow</b>)</span><span style="font-size:0.75em;" ></span></p>

Note:
1. enable debug
2. memory initialization
3. boot to UEFI shell only
4. boot to OS
5. boot to OS w/ security enabled
6. Advanced Feature Selection
7. Not shown  Performance Optimizations

+++?image=assets/images/slides/Slide49.JPG
<!-- .slide: data-background-transition="none" -->
<!-- .slide: data-transition="none" -->
@title[Stages vs. Boot Flow 06]
<p align="right"><span class="gold" >@size[1.1em](<b>Stages vs. Boot Flow</b>)</span><span style="font-size:0.75em;" ></span></p>

Note:
1. enable debug
2. memory initialization
3. boot to UEFI shell only
4. boot to OS
5. boot to OS w/ security enabled
6. Advanced Feature Selection
7. Not shown  Performance Optimizations

---?image=assets/images/slides/Slide50.JPG
@title[Staged Approach by Features]
<p align="right"><span class="gold" >@size[1.1em](<b>Staged Approach by Features</b>)<br></span><span style="font-size:0.75em;" >- Firmware Volume</span></p>

@snap[north-east span-60 ]
<br>
<br><br>
<br><br>
<p style="line-height:80%" align="left" ><span style="font-size:0.9em;" >
Modules organized by Firmware Volumes according to the different boot stages
</span>
<span style="font-size:0.5em; font-family:Consolas;">
</span></p>
@snapend


Note:

HOW is it implemented??

In order to separate modules in different boot stages, the BKM is to Standardize the firmware layout using Firmware Volumes according to the different boot stages


---?image=assets/images/slides/Slide51.JPG
@title[UEFI Firmware Volumes (FV) - Review]
<p align="right"><span class="gold" >@size[1.1em](<b>UEFI Firmware Volumes (FV) - Review</b>)<br></span><span style="font-size:0.75em;" ></span></p>

@snap[north-west span-70 ]
<br>
<p style="line-height:70%" align="left" ><span style="font-size:0.8em;" >
<br>
Platform Initialization - Firmware Volume 
</span></p>

<ul style="list-style-type:disc; line-height:0.7;">
  <li><span style="font-size:0.75em" >Basic storage repository for data and code is the Firmware Volume (FV)  </span> </li>
  <li><span style="font-size:0.75em" >Each FV is organized into a file system, each with attributes </span> </li>
  <li><span style="font-size:0.75em" >One or more Firmware File Sections (FFS) files are combined into a FV  </span> </li>
  <li><span style="font-size:0.75em" >Flash Device may contain one or more FVs </span> </li>
  <li><span style="font-size:0.75em" >.FDF file controls the layout → .FD image(s) </span> </li>
</ul>
<br>
<br>
<p style="line-height:70%" align="left" ><span style="font-size:0.65em;" >
<a href="https://uefi.org/specifications"> PI Spec Vol. 3</a>
</span></p>

@snapend


Note:
In order to separate modules in different boot stage, we standardize the firmware layout. 
First review of EDK II – UEFI Firmware Volumes
Platform Initialization - Firmware Volume 

- Basic storage repository for data and code is the Firmware Volume (FV) 
- Each FV is organized into a file system, each with attributes
- One or more Firmware File Sections (FFS) files are combined into a FV 
- Flash Device may contain one or more FVs.
- .FDF file controls the layout 



- FvPreMemory – The PEIM dispatched before the memory initialization. 
- FvPostMemory – The PEIM dispatched after the memory initialization. 
- FvUefiBoot – The DXE driver supporting UEFI boot, such as boo to UEFI shell. 
- FvOsBoot – The DXE driver supporting UEFI OS boot, such as UEFI Windows. 
- FvSecurity – The security related modules, such as UEFI Secure boot, TPM etc. 
- FvAdvanced – The advanced feature modules, such as UEFI network, IPMI etc. 



---
@title[Standardize FV By Stages]
<p align="right"><span class="gold" >@size[1.1em](<b>Standardize FV By Stages</b>)</span></span></p>


@snap[north-east span-71 ]
<p style="line-height:20%"><br><br><br>&nbsp;</p>

@box[bg-grey-15 text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" ><b>&nbsp; </b><br>&nbsp;</span></p>)
@box[bg-grey-15 text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" ><b>&nbsp; </b><br>&nbsp;</span></p>)
@box[bg-grey-15 text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" ><b>&nbsp; </b><br>&nbsp;</span></p>)
@box[bg-grey-15 text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" ><b>&nbsp; </b><br>&nbsp;</span></p>)
@box[bg-grey-15 text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" ><b>&nbsp; </b><br>&nbsp;</span></p>)
@box[bg-grey-15 text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" ><b>&nbsp; </b><br>&nbsp;</span></p>)
@snapend



@snap[north-west span-30 ]
<p style="line-height:20%"><br><br><br>&nbsp;</p>

@box[bg-blue-pp text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" ><b>Pre-Memory </b><br>&nbsp;</span></p>)
@box[bg-blue-pp text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" ><b>Post-Memory  </b><br>&nbsp;</span></p>)
@box[bg-blue-pp text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" ><b>UEFI Boot </b><br>&nbsp;</span></p>)
@box[bg-blue-pp text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" ><b>OS Boot </b><br>&nbsp;</span></p>)
@box[bg-blue-pp text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" ><b>Security </b><br>&nbsp;</span></p>)
@box[bg-blue-pp text-white rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.7em;" ><b>Advanced </b><br>&nbsp;</span></p>)
@snapend




@snap[north-east span-65 fragment ]
<p style="line-height:20%"><br><br><br><br>&nbsp;</p>
<p style="line-height:32%" align="left"><span style="font-size:0.5em;" ><b>@color[yellow](FvPreMemory) </b>– The PEIM dispatched before the memory initialization. Also included @color[yellow](FSP - FVs) 
<br><br>&nbsp;</span></p>
<p style="line-height:32%" align="left"><span style="font-size:0.5em;" ><b>@color[yellow](FvPostMemory)</b> – The PEIM dispatched after the memory initialization. Also included @color[yellow](FSP - FVs)
<br><br>&nbsp;</span></p>
<p style="line-height:32%" align="left"><span style="font-size:0.5em;" ><b>@color[yellow](FvUefiBoot)</b> – The DXE driver supporting UEFI boot, such as boot to UEFI shell. 
<br><br>&nbsp;</span></p>
<p style="line-height:32%" align="left"><span style="font-size:0.5em;" ><b>@color[yellow](FvOsBoot)</b> – The DXE driver supporting UEFI OS boot, such as UEFI Windows
<br><br>&nbsp;</span></p>
<p style="line-height:32%" align="left"><span style="font-size:0.5em;" ><b>@color[yellow](FvSecurity)</b> – The security related modules, such as UEFI Secure boot, TPM etc. 
<br><br>&nbsp;</span></p>
<p style="line-height:32%" align="left"><span style="font-size:0.5em;" ><b>@color[yellow](FvAdvanced) </b>– The advanced feature modules, such as UEFI network, IPMI etc 
<br>&nbsp;</span></p>
@snapend



Note:

In order to separate modules in different boot stage, BKM to Standardize the firmware layout. 

- FvPreMemory – The PEIM dispatched before the memory initialization. 
- FvPostMemory – The PEIM dispatched after the memory initialization. 
- FvUefiBoot – The DXE driver supporting UEFI boot, such as boo to UEFI shell. 
- FvOsBoot – The DXE driver supporting UEFI OS boot, such as UEFI Windows. 
- FvSecurity – The security related modules, such as UEFI Secure boot, TPM etc. 
- FvAdvanced – The advanced feature modules, such as UEFI network, IPMI etc. 

---
@title[FSP Firmware Volumes ]
<p align="right"><span class="gold" >@size[1.1em](<b>FSP Firmware Volumes </b>)<br></span><span style="font-size:0.75em;" >- Created Pre-Build</span></p>

@snap[north-west span-49 ]
<br>
<br>
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend


@snap[north-east span-48 ]
<br>
<br>
<p style="line-height:50%" align="left" ><span style="font-size:0.75em; ">
@color[#A8ff60](Fsp.fd) Rebased for FVs

</span></p>
@snapend



@snap[north span-29 fragment ]
<br>
<br>
<br>
<br>
@box[bg-green-pp text-black rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.9em;" ><b>FvFspT</b><br><br>&nbsp;</span></p>)
@box[bg-green-pp text-black rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.9em;" ><b>FvFspM</b><br><br>&nbsp;</span></p>)
@box[bg-green-pp text-black rounded my-box-pad2  ](<p style="line-height:60%"><span style="font-size:0.9em;" ><b>FvFspS</b><br><br>&nbsp;</span></p>)
@snapend

@snap[north-east span-35 fragment]
<br>
<br>
<br>
<br>
<br>
@box[bg-grey-15 text-white rounded my-box-pad2  ](<p style="line-height:20%" align="left"><span style="font-size:0.45em;" >&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- Temp Memory<br>&nbsp;</span></p>)
<p style="line-height:20%"><br>&nbsp;</p>
@box[bg-grey-15 text-white rounded my-box-pad2  ](<p style="line-height:20%" align="left"><span style="font-size:0.45em;" >&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- -&gt;FvPreMemorySilicon<br>&nbsp;</span></p>)
<p style="line-height:20%"><br>&nbsp;</p>
@box[bg-grey-15 text-white rounded my-box-pad2  ](<p style="line-height:20%" align="left"><span style="font-size:0.45em;" >&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- -&gt;FvPostMemorySilicon<br>&nbsp;</span></p>)
@snapend


@snap[north-west span-60 ]
<br>
<br>
<p style="line-height:50%" align="left" ><span style="font-size:0.5em; font-family:Consolas;">
&nbsp;MyWorkSpace/<br>&nbsp;&nbsp;
@color[yellow](edk2)/<br>&nbsp;&nbsp;&nbsp;&nbsp;
  - "@color[#FFC000](<font face="Arial">edk2 Common</font>)"<br>&nbsp;&nbsp;
@color[yellow](edk2-platforms)/<br>&nbsp;&nbsp;&nbsp;&nbsp;
  Platform/ "@color[#FFC000](<font face="Arial">Platform</font>)"<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     Intel/MinPlatformPkg<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
       include/  <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
          @color[#A8ff60](flashmapinclude.fdf) <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
          BoardXPkg/ “@color[#FFC000](<font face="Arial">Board</font>)”<br>&nbsp;&nbsp;&nbsp;&nbsp;
  Silicon/ "@color[#FFC000](<font face="Arial">Silicon</font>)"<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     Intel/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
       MinPlatformPkg/<br>&nbsp;&nbsp;
@color[yellow](edk2-non-osi)/<br>&nbsp;&nbsp;&nbsp;&nbsp;
  Silicon/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     Intel/<br>&nbsp;&nbsp;
@color[yellow](FSP)/"@color[#FFC000](<font face="Arial">Silicon</font>)"<br>&nbsp;&nbsp;&nbsp;&nbsp;
   BoardXPkg/<br>&nbsp;&nbsp;&nbsp;&nbsp;
   @color[#A8ff60](<b>Fsp.fd</b>)
</span></p>
@snapend

@snap[south-east span-40 ]
<p style="line-height:40%" align="left" ><span style="font-size:0.5em; ">
Pre-Build w/ <b><font face="Consolas">RebaseAndPatchFspBinBaseAddress.py </font></b>
</span></p>
<br>
@snapend


Note:

Prior to the EDK II build the Tool RebaseAndPatchFspBinBaseAddress.py will rebase the Fsp.Fd using the board Flashmapinclude.fdf file into 3 FVs

- FvFspT
– Temp Memory
- FvFspM
-> FvPreMemorySilicon
- FvFspS
-> FvPostMemorySilicon


---?image=assets/images/slides/Slide54.JPG
@title[FSP APIs in FSP Binary]
<p align="right"><span class="gold" >@size[1.1em](<b>FSP APIs in FSP Binary</b>)<br></span><span style="font-size:0.5em;" >Using Intel<sup>&reg;</sup> FSP w/ EDK II: 
<a href="https://firmware.intel.com/sites/default/files/A_Tour_Beyond_BIOS_Using_the_Intel_Firmware_Support_Package_with_the_EFI_Developer_Kit_II_(FSP2.0).pdf">PDF</a>
</span></p>

Note:
Despite the variability of the FSP binaries, the FSP API caller (aka FSP consumer) could be a generic module to invoke the 5 APIs defined in FSP EAS (External Architecture Specification) to finish silicon initialization. 
5 APIs are:
  TempRamInit, NotifyPhase, FspMemoryInit, TempRamExit, FspSiliconInit 
 
The flow on this slide describes the FSP, with the FSP binary from the “FSP Producer” in green and the platform code that integrates the binary, or the “FSP Consumer”, in blue 


The FSP EAS describes the API interface to the FSP binary that the consumer code will 
invoke, and it also describes the hand off state from the execution of the FSP binary. The latter information is conveyed in Hand-Off Blocks, or HOB’s. The FSP uses many of the data structures defined in the PI Specification including HOBs, Firmware Volumes, Firmware Files, etc. 
The FSP binary can be integrated into any firmware solution, such as UEFI firmware (EDK II)

1. Bootloader provided SEC phase starts executing from Reset Vector 
	a) Switches the mode to 32-bit mode. 
	b) Initializes the early platform as needed. 
	c) Finds FSP-T and calls the TempRamInit() API. The bootloader also has the option to initialize the temporary memory directly, in which case this step and step 2 are skipped. 
2. FSP initializes temporary memory and returns from TempRamInit() API. 
3. Bootloader initializes the stack in temporary memory. 
	a) Initializes the platform as needed. 
	b) Finds FSP-M and calls the FspMemoryInit() API. 
4. FSP initializes memory and returns from FspMemoryInit() API. 
5. Bootloader relocates itself to Memory. 
6. Bootloader calls TempRamExit() API. If Bootloader initialized the temporary memory in step 1.c)… this step and the next step are skipped. 
7. FSP returns from TempRamExit() API. 
8. Bootloader finds FSP-S and calls FspSiliconInit() API. 
9. FSP returns from FspSiliconInit() API. 
10. Bootloader continues and device enumeration. 

11. Bootloader calls NotifyPhase() API with AfterPciEnumeration parameter. 
12. Bootloader calls NotifyPhase() API with ReadyToBoot parameter before transferring control to OS loader. 
13. When booting to a non-UEFI OS, Bootloader calls NotifyPhase() API with EndOfFirmware parameter immediately after ReadyToBoot. 
14. When booting to a UEFI OS, Bootloader calls NotifyPhase() with EndOfFirmware parameter during ExitBootServices. 
Another Note: : If FSP returns the reset required status in any of the API, then bootloader performs the reset. Refer to the Integration Guide for more details on Reset Types. 


---?image=assets/images/slides/Slide55.JPG
@title[Boot Flow with FSP API Mode]
<p align="right"><span class="gold" >@size[1.1em](<b>Boot Flow with FSP API Mode</b>)</span><span style="font-size:0.5em;" ></span></p>

<p style="line-height:40%" align="left" ><span style="font-size:0.75em; ">
5 APIs for FSP
</span></p>

@snap[south-west span-40 ]
<p style="line-height:40%" align="left" ><span style="font-size:0.5em;" >Using Intel<sup>&reg;</sup> FSP w/ EDK II: 
<a href="https://firmware.intel.com/sites/default/files/A_Tour_Beyond_BIOS_Using_the_Intel_Firmware_Support_Package_with_the_EFI_Developer_Kit_II_(FSP2.0).pdf">PDF</a>
</span></p>
@snapend

Note:

The normal boot flow of FSP2.0 is shown on this slide. In normal boot, the SecPlatformLib (sample at https://github.com/tianocore/edk2/tree/master/IntelFsp2WrapperPkg/Library/SecFspWrapperPlatformSecLibSample ), which is linked by the SecCore (https://github.com/tianocore/edk2/tree/master/UefiCpuPkg/SecCore ), calls first FSP API – TempRamInitApi, and then transfers the control to the PeiCore. SecPlatformLib also registers SecTempRamDonePpi (https://github.com/tianocore/edk2/blob/master/IntelFsp2WrapperPkg/Library/SecFspWrapperPlatformSecLibSample/SecTempRamDone.c ) for TempRamExitApi. 

One platform PEIM is responsible to detect the current boot mode and finds some variables (like capsule variable) to finalize the boot mode selection. The FspmWrapperPeim (https://github.com/tianocore/edk2/tree/master/IntelFsp2WrapperPkg/FspmWrapperPeim ) has a dependency on MasterBootModePpi, so after the boot mode is determined, the FspmWrapperPeim is invoked. FspmWrapperPeim gets the UPD data, allocates buffer for UPD override data, then calls UpdateFspmUpdData() to update the UPD data according to platform policy (sample at https://github.com/tianocore/edk2/tree/master/IntelFsp2WrapperPkg/Library/BaseFspWrapperPlatformLibSample ). Then FspmWrapperPeim calls second FSP API – FspMemoryInitApi. Once this API returns, FspmWrapperPeim calls PostFspmHobProcess() to process the initial FSP HOB. (sample at https://github.com/tianocore/edk2/tree/master/IntelFsp2WrapperPkg/Library/PeiFspWrapperHobProcessLibSample ). FspWrapperHobProcessLib parses resource HOB and installs PEI memory to PEI core. 

Once the PeiCore gets permanent memory, PeiCore does TemporaryRam migration and calls PeiTemporaryRamDonePpi, where TempRamExitApi is called. After that, PeiCore installs PeiMemoryDiscovered. Then the dependency of FspsWrapperPeim (https://github.com/tianocore/edk2/tree/master/IntelFsp2WrapperPkg/FspsWrapperPeim ) is satisfied. FspsWrapperPeim gets the UPD data, allocates buffer for UPD override data, then calls UpdateFspsUpdData() to update the UPD data according to platform policy. (sample at https://github.com/tianocore/edk2/tree/master/IntelFsp2WrapperPkg/Library/BaseFspWrapperPlatformLibSample ). Then FspsWrapperPeim calls FspSiliconInitApi to finish final silicon initialization. Once this API returns, FspsWrapperPeim calls PostFspsHobProcess() to process FSP HOB after silicon initialization. (sample at https://github.com/tianocore/edk2/tree/master/IntelFsp2WrapperPkg/Library/PeiFspWrapperHobProcessLibSample ). Typically, there will be more data in the FSP HOB at this time. Most work in PostFspsHobProcess() is to migrate the HOB data from FSP to FspWrapper. 

Then the PeiCore will continue dispatching the final PEIMs and jump into the DxeCore. Then the DxeCore launches FspWrapperNotifyDxe (https://github.com/tianocore/edk2/tree/master/IntelFsp2WrapperPkg/FspWrapperNotifyDxe ). FspWrapperNotifyDxe registers a callback function for the last FSP API – FspNotifyApi, for AfterPciEnumeration, ReadyToBoot, and EndOfFirmware. 

---?image=assets/images/slides/Slide56.JPG
@title[FSP 2.1 Dispatch Mode Boot Flow]
<p align="right"><span class="gold" >@size[1.1em](<b>FSP 2.1 Dispatch Mode Boot Flow</b>)</span><span style="font-size:0.5em;" ></span></p>

<p style="line-height:40%" align="left" ><span style="font-size:0.5em; ">
<b><font face="Consolas">gIntelFsp2WrapperTokenSpaceGuid</font>.@color[yellow](<font face="Consolas">PcdFspModeSelection</font>)</b>	0 - dispatch, 1 – API
</span></p>

@snap[south-west span-40 ]
<p style="line-height:40%" align="left" ><span style="font-size:0.6em; ">
<a href="https://www.intel.com/FSP">FSP Spec 2.1</a>
</span></p>
@snapend

Note:

FSP 2.1 introduces Dispatch Mode Boot Flow

The Pcd - gIntelFsp2WrapperTokenSpaceGuid.PcdFspModeSelection	0 - dispatch, 1 – API (default)  is used

Dispatch mode is an optional boot flow intended to enable FSP to integrate well in to UEFI bootloader implementations. Implementation of this boot flow necesitates that the underlying FSP implementation uses the Pre-EFI Initialization (PEI) environment defined in the PI Specification. 

Blue blocks are from the FSP binary and green blocks are from the bootloader. Blocks with mixed colors indicate that both bootloader and FSP modules are dispatched during that phase of the boot flow. 
Dispatch mode is intended to implement a boot flow that is as close to a standard UEFI boot flow as possible. In dispatch mode, FSP exposes Firmware Volumes (FV) directly to the bootloader. The PEIM in these FV are executed directly in the context of the PEI environment provided by the bootloader. FSP-T, FSP-M, and FSP-S could contain one or multiple FVs. The exact FVs layout will be described in the Integration Guide. In dispatch mode, the PPI database, PCD database, and HOB list are shared between the bootloader and the FSP. 
In dispatch mode, the NotifyPhase() API API is not used. Instead, FSP-S contains DXE drivers that implement the native callbacks on equivalent events for each of the NotifyPhase() invocations. 


---?image=assets/images/slides/Slide57.JPG
@title[ Platform Hooks Section]
<br><br><br><br><br><br><br>
### <span class="gold"  >&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Platform Hooks</span>
<span style="font-size:0.9em" >&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Using EDK II Libraries</span>


@snap[south-east span-25 fragment]
![how](/assets/images/How_text.png)
<br>
<br>
<br>
@snapend

Note:

How? – by using EDK II Libraries for Platform Hooks


---?image=assets/images/slides/Slide58.JPG
@title[EDK II Libraries w/ Platform Hooks]
<p align="right"><span class="gold" >@size[1.1em](<b>EDK II Libraries w/ Platform Hooks </b>)</span><span style="font-size:0.75em;" ></span></p>



@snap[north-east span-80 fragment ]
<br>
<br>
<br>
<br>
<p style="line-height:75%" align="left"><span style="font-size:0.9em">@color[yellow](DSC maps library class to library-instances)</span></p>
<br>
@snapend


@snap[north-east span-80 fragment ]
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<p style="line-height:70%" align="left"><span style="font-size:0.9em">Syntax in DSC File</span><br>
<span style="font-size:0.55em; font-family:Consolas;">
&nbsp;&nbsp;[libraryclasses] <br>
&nbsp;&nbsp;LibraryClassName|Path/To/@color[cyan](LibInstanceNameInstance1).inf  </span> </p>  
<br>
@snapend


@snap[south span-85 fragment ]
@box[bg-purple-pp text-white rounded   my-box-pad2 ](<span style="font-size:01.0em" >Search INF files for string:&nbsp;&nbsp; "<font face="Consolas">LIBRARY_CLASS  =</font>"<br>&nbsp;</span>)
<br>
@snapend


Note:

DSC maps library-class to library-instances
Global, by type, individual override
Effected by size, speed, built in, binary distribution 
PCI example
PciCfg – Protocol function call by GUID (PEI)
PciRootBridgeIo – Protocol function call by GUID (DXE)
Syntax in DSC file
	[libraryclasses] 
	LibraryClassName|Path/To/LibInstanceNameInstance1.inf
	
	
also note, Workspace relative paths!

Check for existing library instances.

Search INF for string: LIBRARY_CLASS  =


---?image=assets/images/slides/Slide59.JPG
@title[Library Classes Section in DSC]
<p align="right"><span class="gold" >@size[1.1em](<b>Library Classes Section in DSC </b>)</span><span style="font-size:0.75em;" ></span></p>

<p style="line-height:70%" ><span style="font-size:0.9em; font-weight: bold;" ><font face="Consolas">DebugLib</font> class example </span></p>
<br>
```
 [LibraryClasses]
     DebugLib|MdePkg/Library/BaseDebugLibNull/BaseDebugLibNull.inf
     • • •
 [LibraryClasses.common.DXE_CORE]
         • • •
    DebugLib|IntelFrameworkModulePkg/Library/PeiDxeDebugLibReportStatusCode/
           PeiDxeDebugLibReportStatusCode.inf  
        • • •
 [LibraryClasses.common.DXE_SMM_DRIVER]
    DebugLib|MdePkg/Library/BaseDebugLibNull/BaseDebugLibNull.inf

```
<hr>
```
 [Components]
      • • •
 MyPath/MyModule.inf {
  <LibraryClasses>
    DebugLib|MdePkg/Library/BaseDebugLibSerialPort.inf
 }
 
``` 


@snap[north-east span-35 fragment ]
<br>
<br>
<br>
<br>
@box[bg-purple-pp text-white my-box-pad2  ](<span style="font-size:0.75em;"><b> Library Class Section</b></span>)
<br>
<br>
<br>
<br>
<br>
@box[bg-green-pp text-white my-box-pad2  ](<span style="font-size:0.75em;"><b> Components Section</b></span>)
@snapend

Note:

This is an example!!!

Build the slide with each libraryclass


Library instances selected in the DSC help with porting

Only one instance of each named library class may be linked to a given module

---
@title[Platform Initialization Board Hook Modules]
<p align="right"><span class="gold" >@size[1.1em](<b>Platform Initialization Board Hook Modules</b>)</span><span style="font-size:0.75em;" ></span></p>


@snap[north-west span-40 ]
<br>
<br>
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend

@snap[north-east span-58 ]
<br>
<br>
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br>&nbsp;</span></p>)
@snapend


@snap[north-east span-12 ]
<br>
![hook](/assets/images/hook.png)
@snapend

@snap[north-west span-60 ]
<br>
<br>
<p style="line-height:50%" align="left" ><span style="font-size:0.5em; font-family:Consolas;">
<br>&nbsp;&nbsp;
MinPlatformPkg/<br>&nbsp;&nbsp;&nbsp;&nbsp;
  Include/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     Library/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
	   @color[yellow](BoardnitLib.h)<br>&nbsp;&nbsp;&nbsp;&nbsp;
  Library/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  . . .<br><br>&nbsp;&nbsp;&nbsp;&nbsp;
  @color[yellow](PlatformInit)/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    PlatformInitPei/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
	  PlatformInitPreMem/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
	  PlatformInitPostMem/<br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    PlatformInitDxe/<br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    PlatformInitSmm/  <br>&nbsp;&nbsp;&nbsp;&nbsp;
</span></p>
@snapend

@snap[north-east span-55 ]
<br>
<br>
<p style="line-height:50%" align="left" ><span style="font-size:0.5em; font-family:Consolas;">
<br>
BoardDetect()<br>
BoardDebugInit()<br>
BoardBootModeDetect()<br>
BoardInitBeforeMemoryInit()<br>
BoardInitBeforeTempRamExit()<br>
BoardInitAfterTempRamExit()<br>
BoardInitAfterMemoryInit()<br>
BoardInitBeforeSiliconInit()<br>
<br><br><br><br>
BoardInitAfterPciEnumeration()<br>
BoardInitReadyToBoot()<br>
BoardInitEndOfFirmware()<br>
</span></p>
@snapend


@snap[north-east span-18 fragment]
<br><br><br><br><br>
<p style="line-height:70%" ><span style="font-size:01.1em; font-weight: bold;" >@color[yellow](PEI)</span></p>
<p style="line-height:40%" ><span style="font-size:0.9em; font-weight: bold;" ><br><br><br><br><br><br><br><br>&nbsp;</span></p>
<p style="line-height:70%" ><span style="font-size:01.1em; font-weight: bold;" >@color[yellow](DXE)</span></p>
@snapend




Note:

The PlatformInit folder (Intel/MinPlatformPkg/PlatformInit) - PlatformInitPei, PlatformInitDxe and PlatformInitSmm control the platform initialization flow. Because this flow needs to involve the board initialization,  there is a set of  board hook points defined in BoardInitLib (MinPlatformPkg/Include/Library/BoardInitLib.h) 

---
@title[Platform Initialization Board Hook Modules]
<p align="right"><span class="gold" >@size[1.1em](<b>Platform Initialization Board Hook Modules</b>)</span><span style="font-size:0.75em;" ></span></p>


@snap[north-west span-40 ]
<br>
<br>
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@snapend

@snap[north-east span-58 ]
<br>
<br>
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br><br><br><br><br><br><br><br>&nbsp;</span></p>)
@box[bg-black text-white rounded my-box-pad2  ](<p style="line-height:60% "><span style="font-size:0.9em;" ><br><br><br><br>&nbsp;</span></p>)
@snapend


@snap[north-east span-12 ]
<br>
![hook](/assets/images/hook.png)
@snapend


@snap[north-west span-60 ]
<br>
<br>
<p style="line-height:50%" align="left" ><span style="font-size:0.5em; font-family:Consolas;">
<br>&nbsp;&nbsp;
MinPlatformPkg/<br><br>&nbsp;&nbsp;&nbsp;&nbsp;
  . . .<br><br>&nbsp;&nbsp;&nbsp;&nbsp;
  PlatformInit/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    PlatformInitPei/<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
	  @color[yellow](PlatformInitPreMem)/
	  <br><br><br><br>
	  <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
	  @color[yellow](PlatformInitPostMem)/
</span></p>
@snapend

@snap[north-east span-55 ]
<br>
<br>
<p style="line-height:50%" align="left" ><span style="font-size:0.5em; font-family:Consolas;">
<br>
@color[yellow](PlatformInitPreMem)/<br>&nbsp;&nbsp;
BoardDetect()<br>&nbsp;&nbsp;
BoardDebugInit()<br>&nbsp;&nbsp;
BoardBootModeDetect()<br>&nbsp;&nbsp;
BoardInitBeforeMemoryInit()<br>&nbsp;&nbsp;
. . . <br><br>&nbsp;&nbsp;
<font face="Arial">Notify call back </font><br>&nbsp;&nbsp;
BoardInitAfterMemoryInit()
<br><br><br><br><br>
@color[yellow](PlatformInitPostMem)/<br>&nbsp;&nbsp;

BoardInitBeforeSiliconInit()<br>&nbsp;&nbsp;
. . . <br>&nbsp;&nbsp;
BoardInitAfterSiliconInit()<br>
</span></p>
@snapend


@snap[north-east span-18 fragment]
<br><br><br><br><br><br><br><br>
<p style="line-height:70%" ><span style="font-size:01.1em; font-weight: bold;" ><br><br><br>@color[yellow](PEI)</span></p>

@snapend




Note:

The PlatformInit folder (Intel/MinPlatformPkg/PlatformInit) - PlatformInitPei, PlatformInitDxe and PlatformInitSmm control the platform initialization flow. Because this flow needs to involve the board initialization,  there is a set of  board hook points defined in BoardInitLib (MinPlatformPkg/Include/Library/BoardInitLib.h) 

---?image=assets/images/slides/Slide5.JPG
@title[EDK II Open Platform Summary Section]
<br>
<p align="left"><span class="gold" >@size[1.1em](<b>EDK II Open Platform <br>Summary</b>)</span></span></p>

@snap[north-east span-35 ]
<br>
<br>

<ul style="list-style-type:disc; line-height:0.7;">
  <li><span style="font-size:0.65em" >Minimal /Full BIOS </span> </li>
  <li><span style="font-size:0.65em" >Feature ON/OFF </span> </li>
  <ul style="list-style-type:disc; line-height:0.6;">
  <li><span style="font-size:0.6em" >SMBIOS</span> </li>
  <li><span style="font-size:0.6em" >TPM </span> </li>
  <li><span style="font-size:0.6em" >Secure Boot </span> </li>
  <li><span style="font-size:0.6em" >. . . </span> </li>
  </ul>
</ul>
@snapend

@snap[south-west span-30 ]
<br>
<ul style="list-style-type:disc; line-height:0.7;">
  <li><span style="font-size:0.65em" >Setup Variable</span> </li>
  <li><span style="font-size:0.65em" >PCD </span> </li>
  <li><span style="font-size:0.65em" >Policy Hob/PPI/Protocol </span> </li>
</ul>
<br>
<br>
<br>
@snapend

@snap[south-east span-33 ]
<br>
<ul style="list-style-type:disc; line-height:0.7;">
  <li><span style="font-size:0.65em" >GPIO </span> </li>
  <li><span style="font-size:0.65em" >SIO </span> </li>
  <li><span style="font-size:0.65em" >ACPI </span> </li>
  <li><span style="font-size:0.65em" >Stages </span> </li>
</ul>
<br>
<br>
<br>
<br>
@snapend



Note:

In order to provide suggestions on the problem statements earilier, we need to focus on the following four areas: 

- Feature. How does a BIOS provide the feature selection option to a developer? 
- Configuration. From which interface can a platform module get the configuration data? 
- Porting. Where are the modules to be ported for a new board? 
- Tree Structure. What does an EDKII platform package look like? 

---  
@title[summary]
<BR>
### <p align="center"<span class="gold"   >Summary </span></p><br>

<!---  Add bullets using https://fontawesome.com/cheatsheet certificate
-->
<ul style="list-style-type:none">
 <li>@fa[certificate gp-bullet-green]<span style="font-size:0.9em">&nbsp;&nbsp;Explain the EDK II Open board platforms <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;infrastructure  & focus areas</span> </li>
 <li>@fa[certificate gp-bullet-cyan]<span style="font-size:0.9em">&nbsp;&nbsp;Describe Intel® FSP with  the EDK II Open board<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;platforms </span></li>
</ul>



---?image=assets/images/gitpitch-audience.jpg
@title[Questions]
<br>
![Questions](/assets/images/questions.JPG) 

---
@title[return to main]
<p align="center"><span class="gold"   >@size[1.2em](<b>Return to Main Training Page</b>)</span></p>
<br>
<br>
<br>
<br>
<br>
<p align="center"><span style="font-size:0.9em">Return to Training Table of contents for next presentation <a href="https://github.com/tianocore-training/Tianocore_Training_Contents/wiki#schedule--outline">link</a></span></p>

@snap[north span-30 ]
<br>
<br>
<br>
<a href="https://github.com/tianocore-training/Tianocore_Training_Contents/wiki#schedule--outline">
![trainingLogo](/assets/images/returnTrainingLogo.png)</a>
@snapend


---?image=assets/images/gitpitch-audience.jpg
@title[Logo Slide]
<br><br><br>
![Logo Slide](/assets/images/TianocoreLogo.png =10x)


---
@title[Acknowledgements]
#### <p align="center"><span class="gold"   >Acknowledgements</span></p>

```c++
/**
Redistribution and use in source (original document form) and 'compiled' forms (converted
to PDF, epub, HTML and other formats) with or without modification, are permitted provided
that the following conditions are met:

Redistributions of source code (original document form) must retain the above copyright 
notice, this list of conditions and the following disclaimer as the first lines of this 
file unmodified.

Redistributions in compiled form (transformed to other DTDs, converted to PDF, epub, HTML
and other formats) must reproduce the above copyright notice, this list of conditions and 
the following disclaimer in the documentation and/or other materials provided with the 
distribution.

THIS DOCUMENTATION IS PROVIDED BY TIANOCORE PROJECT "AS IS" AND ANY EXPRESS OR IMPLIED 
WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND 
FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL TIANOCORE PROJECT BE 
LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES 
(INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, 
DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, 
WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) 
ARISING IN ANY WAY OUT OF THE USE OF THIS DOCUMENTATION, EVEN IF ADVISED OF THE POSSIBILITY 
OF SUCH DAMAGE.

Copyright (c) 2019, Intel Corporation. All rights reserved.
**/

```
