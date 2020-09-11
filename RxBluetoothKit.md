## RxSwift code sample of a project, which update the firmware of BLE wearable device

```
func updateFirmware(firmware: String) -> Observable<Result<Int, Error>> {
        turnLED()
        startOTA()
        
        startScanning().subscribe(
            onNext: { [weak self] peripheral in
                switch peripheral {
                case .success(let value):
                    let device = value
                    if device.peripheral.name?.contains("Dfu") == true {
                        let path = Bundle.main.path( forResource: firmware, ofType: "zip")
                        let url = URL(fileURLWithPath: path!)
                        let selectedFirmware = DFUFirmware(urlToZipFile:url)
                        let initiator = DFUServiceInitiator().with(firmware: selectedFirmware!)
                        initiator.delegate = self // - to be informed about current state and errors
                        initiator.progressDelegate = self // - to show progress bar
                        let _ = initiator.start(target: device.peripheral.peripheral)
                        
                        self?.stopScannig()
                    }
                case .error(let error):
                    let strError = (error as! RxError).debugDescription
                    self?.firmwareSubject.onNext(Result.error(strError))
                }},
            onError: { [weak self] error in
                print("Scan Error: \(error)")
                let strError = (error as! RxError).debugDescription
                self?.firmwareSubject.onNext(Result.error(strError))
            }).disposed(by: disposeBag)
        
        return firmwareSubject.asObserver()
    }
    ```
