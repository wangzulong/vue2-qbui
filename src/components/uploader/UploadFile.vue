<style scoped lang="scss">
.upload-file-details{
	display: inline-block;
}
.upload-file-preview-container {
	position: absolute;
	top:10px;
	left:10px;
}
.sendbtn {
	position: absolute;
	bottom:10;
	right:0;
}
.upload-file{
	position: relative;
	.inputfile {
		opacity: 0;
		overflow: hidden;
		z-index: -1;
		&+ label {
			cursor: pointer;
		}
		&:focus + label, & + label:hover{
			color: #30343b;
		}
	}

	//Fade
	.fade-enter-active, .fade-leave-active {
		transition: opacity .5s
	}
	.fade-enter, .fade-leave-active {
		opacity: 0
	}
}

</style>

<template>
	<div class="upload-file" :class="{'upload-file-dragdrop': isDragDrop, 'is-dragover': isDragOver}" :id="`upload-file${itemIndex}`">
		<input type="file" name="uploadFile" class="inputfile" :id="`uploadFile${itemIndex}`" :accept="acceptedFormats.join(',')" @change="changed($event)" multiple>
		<label :for="`uploadFile${itemIndex}`">
			<template v-if="!isDragOver && files.length <=	 0"><div v-html="message"></div></template>
			<template v-else><div v-html="dragOverMessage"></div></template>
		</label>
		<transition name="fade">
			<div :id="`upload-file-preview-container${itemIndex}`" class="upload-file-preview-container" v-show="files.length > 0">
				<template v-for="file in files">
					<div class="upload-file-details">
						<div class="upload-file-thumbnail" :class="[`upload-file-${getFileExtension(file)}`]"></div>
						<div class="upload-file-details-body">
							<span class="upload-file-filename">{{file.name.substring(0, 24)}}</span>
	<!--						<button type="button" class="upload-file-remove-file sl-btn" @click="onRemoveFile(file)" v-html="cancelButton"></button> -->
							<i class="icon sumscope-icon icon-close" @click="onRemoveFile(file)" style="cursor:pointer;margin-right:20px;font-size: 16px; color: #30343b;"></i>
						</div>
					</div>
				</template>
				<div class="upload-file-message">{{this.uploadedMessage}}</div>
			</div>
		</transition>
		<button v-if="showButton" v-show="files.length > 0" type="button" class="button sl-btn sendbtn" :class="buttonClass" @click="send">{{buttonTitle}}</button>
	</div>
</template>

<script type="text/babel">

export default {
	name: "UploadFile",

	data(){
		return {
			files: [],
			isDragOver: false
		}
	},

	props: {
		message: {
			type: String,
			required: false,
			default: "选择文件"
		},
		dragOverMessage: {
			type: String,
			required: false,
			default: "拖放文件到这里"
		},
		uploadedMessage: {
			type: String,
			required: false,
			default: ""
		},
		showButton: {
			type: Boolean,
			required: false,
			default: false
		},
		cancelButton: {
			type: String,
			required: false,
			default: "Cancel"
		},
		buttonTitle: {
			type: String,
			required: false,
			default: "开始上传"
		},
		buttonClass: {
			type: Array,
			required: false,
			default() {
				return []
			}
		},
		itemIndex: {
			type: String,
			required: false,
			default: "0"
		},
		isDragDrop:{
			type: Boolean,
			required: false,
			default: false
		},
		acceptedFormats: {
			type: Array,
			required: false,
			default() {
				return []
			}
		}
	},
	computed: {
		isAdvancedUpload(){
			var div = document.createElement("div")
			return (('draggable' in div) || ('ondragstart' in div && 'ondrop' in div)) && 'FormData' in window && 'FileReader' in window
		}
	},
	mounted(){
		if (this.isDragDrop && this.isAdvancedUpload) {
			//Active dragdrop
			this.$el.addEventListener('dragover', this.dragOver, false)
			this.$el.addEventListener('dragenter', this.dragEnter, false)
			this.$el.addEventListener('dragleave', this.dragLeave, false)
			this.$el.addEventListener('drop', this.drop, false)
		}
	},
	methods: {
		changed(e){
			e.preventDefault()
			this.onAddFile(false, e)
		},
		send(){
			this.$emit("send-file", this.files)
		},
		dragOver(e){
			e.preventDefault()
			this.isDragOver = true
		},
		dragEnter(e){
			this.isDragOver = true
		},
		dragLeave(e){
			this.isDragOver = false
		},
		drop(e){
			e.preventDefault()
			this.isDragOver = false
			this.onAddFile(true, e)
		},
		onAddFile(drop, e){
			let files = (drop) ? e.dataTransfer.files : e.target.files
			for (var index in files) {
				let file = files[index]
				if (typeof file == "object") {
					if (this.acceptedFormats.length > 0) {
						this.files.push(file)
					}else{
						if (this.acceptedFormats.indexOf(file.type) > -1) {
							this.files.push(file)
						}else{
							this.files = false
						}
					}
				}
			}
			this.$emit('changed-files', this.files)
		},
		onRemoveFile(e){
			let indexOfFile = this.files.indexOf(e)
			if (indexOfFile > -1) {
				this.files.splice(indexOfFile, 1)
				this.$emit('changed-files', this.files)
			}
		},
		getFileExtension(file){
			let arrFileName = file.name.split(".")
			let fileExtension = arrFileName[arrFileName.length - 1]
			return fileExtension.toLowerCase()
		},
	}
}
</script>
