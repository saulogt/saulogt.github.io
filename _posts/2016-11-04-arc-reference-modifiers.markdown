---
layout: post
title:  "Studies about reference cycles in swift closures - strong, weak and unowned references"
date:   2016-11-06 10:00:00
categories: []
tags: [ iOS, XCode, Closure, ARC, Swift ]
#image: "/images/2016/11/testbuildfailed.png"
comments: true
published: false
---





/*
     Block : instance variable
     Self : unowned reference
     */
    @IBAction func startUnowned(_ sender: Any) {

        self.block1 = { [unowned self] in
            sleep(2);

            DispatchQueue.main.async {
                self.title = "Unowned"
            }
            print("Done 1!");
        }


        DispatchQueue.global().async(execute: block1!);
    }

    /*
     Block : instance variable
     Self : weak reference
     */
    @IBAction func startWeak(_ sender: Any) {

        block2 = { [weak self] in
            sleep(2);

            DispatchQueue.main.async {
                self?.title = "Weak"
            }
            print("Done 2!");
        }

        DispatchQueue.global().async(execute: self.block2!);

    }

    /*
     Block : instance variable
     Self : strong reference
     */
    @IBAction func startStrong(_ sender: Any) {

        self.block3 =
            {
            sleep(2);

            DispatchQueue.main.async {
                self.title = "Strong"
            }
            print("Done 3!");
        }


        DispatchQueue.global().async(execute: block3!);


    }

    /*
     Block : local variable
     Self : unowned reference
     */
    @IBAction func startStrongLocalBlock(_ sender: Any) {
        let block =
            {
                sleep(2);

                DispatchQueue.main.async {
                    self.title = "Strong (Local)"
                }
                print("Done 4!");
        }


        DispatchQueue.global().async(execute: block);


    }


    deinit {
        print("ViewController deinit");
    }
