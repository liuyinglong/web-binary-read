<template>
	<div class="wrapper flexColumn">
		<header>
			<h1>二进制文件读取</h1>
		</header>
		<div class="flex flexAuto">
			<div class="fileListBox scrollBox">
				<div class="fileInput">
					<input id="fileInput" type="file" multiple="multiple" v-on:change="fileChange($event)">
					<div class="addFile" v-on:click="addFile">添加文件</div>
				</div>
				<ul class="fileList">
					<li v-for="(file,index) in fileList"
						v-bind:class="{active:fileActiveIndex===index}"
						v-on:click="changeActiveIndex(index)">
						<h2>{{file.name}}</h2>
					</li>
				</ul>
			</div>
			<div class="flexAuto flex FJustifyBetween mainBox">
				<template v-if="activeFile">
					<div class="scrollBox binaryBox">
						<div class="boxAbsolute flex">
							<pre class="lineNo" v-html="addressLineNo"></pre>
							<pre class="binaryData" v-html="binaryString"></pre>
						</div>
					</div>

					<div class="fileInfoBox flexAuto">
						<p>文件大小：{{activeFile.size}}</p>
						<p>文件类型：{{activeFile.type}}</p>
						<p>开始地址：{{binaryStart}}</p>
						<p>结束地址：{{binaryEnd}}</p>
						<div class="readEventBox">
							<p>调整读取位置：</p>
							<div>
								<span>开始地址：</span>
								<label><input type="text" v-model.number="binaryStartEdit"
											  placeholder="开始地址,从0开始"/></label>
							</div>
							<div>
								<span>结束地址：</span>
								<label><input type="text" v-model.number="binaryEndEdit"
											  v-bind:placeholder="`结束地址,最大为`+(activeFile.size-1)"/></label>
								<button v-on:click="read">读取</button>
							</div>
						</div>
						<div class="addressBox">
							<div class="address"
								 v-bind:style="{left:binaryStartAddress,width:binarySizeProportion}"></div>
						</div>
					</div>
				</template>
			</div>
		</div>
	</div>
</template>


<script>
    export default {
        data() {
            return {
                fileList: [],
                NoArray: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, "A", "B", "C", "D", "E", "F"],
                binaryString: "",
                addressLineNo: "",
                fileActiveIndex: 0,
                binaryStart: 0,
                binaryEnd: 0,
                binaryStartEdit: "",
                binaryEndEdit: "",
                activeFile: "",
                binaryStartAddress: "",
                binarySizeProportion: ""
            }
        },
        computed: {},
        methods: {
            async fileChange(e) {
                this.fileList =Object.keys(e.target.files).map(function (item) {
                    return e.target.files[item]
                }).concat(this.fileList)
                this.fileActiveIndex = 0
                let binaryEnd = this.fileList[this.fileActiveIndex].size <= 2048 ? this.fileList[this.fileActiveIndex].size : 2048
                await this.getBinaryString({start: 0, end: binaryEnd})
            },
            async changeActiveIndex(index){
                this.fileActiveIndex = index
                let binaryEnd = this.fileList[this.fileActiveIndex].size <= 2048 ? this.fileList[this.fileActiveIndex].size : 2048
                await this.getBinaryString({start: 0, end: binaryEnd})
			},
            read() {
                if (typeof this.binaryStartEdit === "undefined" || typeof this.binaryEndEdit === "undefined") {
                    alert("请输入")
                    return
                }
                if (this.binaryEndEdit <= this.binaryStartEdit) {
                    alert("结束地址不能大于等于开始地址")
                    return
                }

                this.binaryEndEdit = this.binaryEndEdit <= this.activeFile.size - 1 ? this.binaryEndEdit : this.activeFile.size - 1

                this.getBinaryString({
                    start: this.binaryStartEdit,
                    end: this.binaryEndEdit,
                })
            },

            addFile() {
                this.$el.querySelector("#fileInput").click()
            },
            sliceBufferArray(array, step = 1, {start, end}) {
                let resultStr = ""
                let addressLineNo = ""
                for (let i = 0, len = array.length; i < len; i += step) {
                    let formatStr = ""
                    array.slice(i, i + step).forEach((item, index) => {
                        let binary = Number(item).toString(16).toUpperCase()
                        binary = binary.length === 2 ? binary : "0" + binary
                        formatStr = formatStr + binary + "  "
                    })
                    addressLineNo = addressLineNo + ((start + i) / 16).toString(16) + "</br>"
                    resultStr = resultStr + formatStr + "</br>"
                }
                return {resultStr, addressLineNo}
            },

            async getBinaryString({start, end}) {
                this.activeFile = this.fileList[this.fileActiveIndex]
                let fileArrayBuffer = await this.readFile(this.activeFile, {start, end})
                let uInt8Array = new Uint8Array(fileArrayBuffer)
                let {addressLineNo, resultStr} = this.sliceBufferArray(uInt8Array, 16, {start, end})
                this.addressLineNo = addressLineNo
                this.binaryString = resultStr
                this.binaryStart = start
                this.binaryEnd = end
                //开始位置百分比
                this.binaryStartAddress = (100 * start / this.activeFile.size) + "%"
                this.binarySizeProportion = (100 * (end - start) / this.activeFile.size) + "%"
            },

            readFile(fileBlob, {start, end, progress}) {
                return new Promise((resolve, reject) => {
                    let reader = new FileReader()
                    let blob
                    if (typeof start !== "undefined") {
                        blob = fileBlob.slice(start, end)
                    } else {
                        blob = fileBlob
                    }
                    reader.addEventListener("load", (e) => {
                        resolve(e.target.result)
                    })
                    reader.addEventListener("error", (error) => {
                        reject(error)
                    })
                    reader.addEventListener("progress", (e) => {
                        progress && progress(e)
                    })
                    reader.readAsArrayBuffer(blob)
                })
            }
        },
        filters: {
            bit(val) {
                val = val.toString(16).toUpperCase()

            }
        }
    }
</script>

<style lang="scss" scoped="">
	@import "../../../style/scss/v";

	header {
		h1 {
			height: 80px;
			line-height: 80px;
			font-size: 24px;
			text-align: center;
			color: #6897BB;
			box-shadow: 0 0 3px rgba(0, 0, 0, .6);
		}
	}

	.fileListBox {
		width: 250px;
		background: #3C3F41;
		overflow: auto;
		.fileInput {
			input {
				display: none;
			}
			.addFile {
				width: 100%;
				height: 50px;
				line-height: 50px;
				border: none;
				outline: none;
				cursor: pointer;
				background: #0D293E;
				color: #a5ba94;
				font-size: 14px;
				padding-left: 15px;
			}
		}
		.fileList {
			li {
				font-size: 14px;
				line-height: 1.5;
				padding: 5px 15px;
				box-sizing: border-box;
				cursor: pointer;
				color: #a5ba94;
				border-bottom: 1px solid #515151;
				h2 {
					font-size: 16px;
					margin-bottom: 10px;
				}
			}
			li.active {
				background: #0D293E;
			}
		}
	}

	.mainBox {
		font-size: 14px;
		line-height: 1.6;
		position: relative;
		.boxAbsolute {
			position: relative;
			top: 0;
			left: 0;
			bottom: 0;
		}
		.binaryBox {
			height: 100%;
			position: relative;
			pre {
				margin: 0;
				display: block;
			}
			.lineNo {
				width: 80px;
				padding-right: 15px;
				color: $grayTitle;
				text-align: right;
				border-right: 1px solid #555555;
				background: #313135;
				min-height: 100%;
			}
			.binaryData {
				padding: 0 15px;
				color: #cb602e;
			}
		}
	}

	.fileInfoBox {
		background: #313135;
		padding: 15px;
		color: #b9b9a3;
		.addressBox {
			margin-top: 5px;
			width: 100%;
			height: 20px;
			border: 1px solid $borderColor2;
			position: relative;
			.address {
				position: absolute;
				height: 100%;
				background: $mainColor;
				min-width: 3px;
			}
		}
		.readEventBox {

			input {
				border: none;
				background: none;
				color: $white;
				border-bottom: 1px solid $borderColor2;
			}
			button {
				border: none;
				cursor: pointer;
				outline: none;
			}
		}
	}

</style>